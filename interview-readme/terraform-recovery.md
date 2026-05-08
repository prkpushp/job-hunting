# Terraform State Recovery & Resource Repair Guide (GCS Backend)

This guide explains practical Terraform recovery strategies when dealing with:

- Corrupted resources in infrastructure
- Drift between Terraform state and real infrastructure
- Lost or corrupted Terraform state in GCS backend

It includes real-world DevOps recovery approaches commonly used in production environments.

---

# 🧠 Terraform Core Concept

Terraform operates using three layers:

| Layer | Description |
|---|---|
| Terraform State (GCS backend) | Source of truth mapping |
| Terraform Configuration (`.tf` files) | Desired state |
| Real Infrastructure (GCP/AWS/Azure) | Actual deployed resources |

Any issue can happen in:
- infrastructure
- state file
- configuration

Recovery depends on identifying where the corruption happened.

---

# 🚨 Common Failure Scenarios

| Scenario | State File | Infrastructure |
|---|---|---|
| Resource corrupted | Healthy | Broken |
| Drift detected | Healthy | Modified manually |
| State corrupted/lost | Broken | Healthy |

---

# ✅ OPTION 1: Replace a Specific Corrupted Resource

## 📌 Use Case

A specific resource is corrupted or misconfigured in infrastructure, but Terraform state is healthy.

Examples:
- VM inaccessible
- corrupted instance
- failed startup script
- broken deployment

---

## 🔧 Command

```bash
terraform state list
```
	google_compute_network.vpc_network
	
	google_compute_subnetwork.private_subnet
	
	google_compute_firewall.allow_ssh
	
	google_compute_instance.vm1
	
	google_compute_disk.vm1_disk

```bash
terraform apply -replace="google_compute_instance.vm1"
```

---

## 🧠 What Happens

Terraform:
- destroys only the targeted resource
- recreates it fresh
- leaves everything else untouched

---

## ✅ Benefits

- minimal impact
- controlled recovery
- production-safe approach

---

## 🧠 Important Concept

This method is preferred over:
- manual deletion
- recreating full infrastructure
- editing state directly

---

# ✅ OPTION 2: Fix Infrastructure Drift

## 📌 Use Case

Terraform state is correct, but infrastructure was modified manually outside Terraform.

Examples:
- changes via cloud console
- firewall rules modified manually
- VM deleted manually
- subnet or tags changed

---

## 🔧 Commands

```bash
terraform plan
terraform apply
```

---

## 🧠 What Happens

Terraform:
- compares real infrastructure vs state
- detects drift
- applies only required corrections

---

## ✅ Key Understanding

Terraform state remains the source of truth.

Terraform will attempt to bring infrastructure back to desired state defined in `.tf` files.

---

# 🚨 OPTION 3: Terraform State File Corrupted or Lost (GCS Backend)

This is the most critical recovery scenario.

---

# 🔹 OPTION 3.1 — Restore State from GCS Versioning (BEST CASE)

## 📌 Use Case

Terraform state file was accidentally deleted or corrupted, but GCS bucket versioning is enabled.

---

## 🔧 Step 1: List Available Versions

```bash
gsutil ls -a gs://<bucket-name>/path/to/state
```

---

## 🔧 Step 2: Restore Previous Version

```bash
gsutil cp gs://bucket/terraform.tfstate#<version> terraform.tfstate
```

---

## 🔧 Step 3: Reinitialize Terraform

```bash
terraform init
terraform plan
```

---

## 🧠 What Happens

- previous healthy state is restored
- Terraform regains infrastructure mapping
- no manual rebuilding required

---

## ✅ Best Practice

Always enable:
- GCS bucket versioning
- remote backend locking

---

# 🔹 OPTION 3.2 — Re-import Resources Using terraform import

## 📌 Use Case

Terraform state is lost only for specific resources, but infrastructure still exists.

---

# 🧠 MOST IMPORTANT CONCEPT

`terraform import` DOES NOT:
- recreate infrastructure
- generate Terraform configuration
- auto-discover resources

It ONLY:
- reconnects existing infrastructure to Terraform state

Think of it as:

> “Attach Terraform state mapping back to an already existing cloud resource.”

---

# 🔧 Import Command Example

```bash
terraform import google_compute_instance.vm1 vm1
```

OR

```bash
terraform import google_compute_instance.vm1 projects/my-project/zones/asia-south1-a/instances/vm1
```

---

# ❓ What Information Is Required?

## ✅ Required

| Required Item | Example |
|---|---|
| Terraform resource address | `google_compute_instance.vm1` |
| Cloud resource ID/name | `vm1` |

---

## ❌ NOT Required During Import

You DO NOT provide:
- subnet details
- startup scripts
- tags
- machine type
- disk configuration
- metadata

---

# 🧠 Why?

Because import only restores:
- Terraform state mapping

NOT:
- full infrastructure configuration

---

# 🔧 What Happens After Import?

Run:

```bash
terraform plan
```

Terraform compares:
- imported infrastructure
vs
- `.tf` configuration

---

# 🧠 Example

Suppose actual VM has:
- startup script
- labels
- machine type

But `.tf` file is missing them.

Then:

```bash
terraform plan
```

will show drift and propose updates.

---

# ⚠️ Important Reality

Import does not fully rebuild state automatically.

You must:
- ensure `.tf` configuration is accurate
- validate everything using `terraform plan`

---

# 🔹 OPTION 3.3 — Full State Loss Recovery (Worst Case)

## 📌 Use Case

Entire Terraform state is missing or corrupted and cannot be restored.

---

# 🔧 Step 1: Reinitialize Backend

```bash
terraform init
```

OR

```bash
terraform init -reconfigure
```

---

# ❓ What Does terraform init -reconfigure Actually Do?

It:
- reconnects Terraform to backend
- refreshes backend configuration
- ignores cached backend settings

---

# ❌ What It DOES NOT Do

It does NOT:
- restore state
- discover resources
- import infrastructure automatically
- rebuild infrastructure mapping

---

# 🧠 Important Understanding

After full state loss:

Terraform has ZERO knowledge of existing infrastructure.

---

# 🔧 What Happens Next?

If you run:

```bash
terraform plan
```

Terraform assumes:

> “No infrastructure exists.”

Therefore it may try to:
- recreate everything
- create duplicate resources
- fail because resources already exist

---

# 🚨 Real Recovery Process

You must manually rebuild state.

---

# 🔧 Step 2: Discover Existing Infrastructure

Example:

```bash
gcloud compute instances list
```

---

# 🔧 Step 3: Import Resources One by One

```bash
terraform import google_compute_instance.vm1 vm1

terraform import google_compute_instance.vm2 vm2
```

---

# 🧠 Recovery Order Matters

Import dependencies first:

1. VPC
2. Subnets
3. Firewall rules
4. Compute instances
5. Load balancers
6. Databases

---

# ⚠️ Important Reality

Terraform cannot safely auto-discover all infrastructure because:
- infrastructure may differ from `.tf`
- some resources may not be Terraform-managed
- configuration may be outdated

Manual imports are required.

---

# 🧠 Mental Model

| Command | Purpose |
|---|---|
| terraform apply -replace | Recreate specific resource |
| terraform plan/apply | Detect and fix drift |
| terraform import | Reattach existing resource to state |
| terraform init | Initialize backend |
| terraform init -reconfigure | Reconfigure backend connection |
| Restore GCS version | Recover previous state |

---

# 🚀 Production Best Practices

## ✅ Always Enable

- GCS versioning
- remote state locking
- CI/CD validation

---

## ✅ Always Run

```bash
terraform plan
```

before:

```bash
terraform apply
```

---

## ✅ Prefer

```bash
terraform apply -replace
```

instead of:
- manual deletion
- direct cloud console modifications

---

## ❌ Avoid

- manual cloud console changes
- editing state directly
- deleting state files manually

---

# 🚀 Interview-Ready Summary

## ❓ “How do you recover Terraform state corruption?”

### ✅ Suggested Answer

> “If a single resource is corrupted, I use `terraform apply -replace` to safely recreate only that resource. If infrastructure drift occurs due to manual changes, I use `terraform plan` and `terraform apply` to 
