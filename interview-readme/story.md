# 📘 DevOps/SRE/Cloud Engineer Interview Story Library

> 🎯 Purpose: Ready-to-use production + incident stories for DevOps / SRE interviews  
> 💡 Format: Simple 3-step structure (Situation → Action → Result)  
> 🚀 Use: Speak directly in interviews

---

# 🚀 1. Production Deployment Failure (Rollback Story)

## 🧩 Situation
A deployment went live but application stopped responding after release.

## ⚙️ Action
- Checked CI/CD pipeline logs  
- Found configuration mismatch in environment variables  
- Triggered rollback to previous stable version  

## ✅ Result
Service restored within 10 minutes with no data loss.

---

# 🚀 2. High CPU Usage Incident

## 🧩 Situation
A production VM showed 100% CPU usage.

## ⚙️ Action
- Used monitoring tools and `top` command  
- Found infinite loop in application process  
- Restarted service and added resource limits  

## ✅ Result
System stabilized and performance normalized.

---

# 🚀 3. DNS Resolution Failure

## 🧩 Situation
Service was unreachable due to DNS issue.

## ⚙️ Action
- Checked DNS records and resolver config  
- Found incorrect A record mapping  
- Fixed DNS entry and flushed cache  

## ✅ Result
Service became reachable again.

---

# 🚀 4. Load Balancer Misrouting Issue

## 🧩 Situation
Traffic was routed to wrong backend servers.

## ⚙️ Action
- Checked internal load balancer configuration  
- Found incorrect health check path  
- Fixed configuration and re-registered instances  

## ✅ Result
Traffic routing was restored correctly.

---

# 🚀 5. SSH Access Failure

## 🧩 Situation
Team was unable to SSH into production servers.

## ⚙️ Action
- Checked security groups and firewall rules  
- Found port 22 blocked after change  
- Restored SSH access rule  

## ✅ Result
SSH access was restored.

---

# 🚀 6. Disk Space Full Issue

## 🧩 Situation
Production server disk became full due to logs.

## ⚙️ Action
- Identified large log files  
- Cleared old logs and rotated active logs  
- Implemented log rotation policy  

## ✅ Result
System recovered and issue prevented in future.

---

# 🚀 7. CI/CD Pipeline Failure

## 🧩 Situation
Deployment pipeline failed during build stage.

## ⚙️ Action
- Checked GitHub Actions logs  
- Found dependency version mismatch  
- Fixed package version and re-triggered pipeline  

## ✅ Result
Pipeline executed successfully.

---

# 🚀 8. Kubernetes Pod CrashLoopBackOff

## 🧩 Situation
Pods were continuously restarting in Kubernetes.

## ⚙️ Action
- Checked pod logs using `kubectl logs`  
- Found missing environment variable  
- Updated deployment configuration  

## ✅ Result
Pods became stable and running.

---

# 🚀 9. API Latency Spike

## 🧩 Situation
API response time suddenly increased.

## ⚙️ Action
- Checked monitoring dashboards  
- Found database query bottleneck  
- Added indexing and optimized query  

## ✅ Result
Latency reduced and performance improved.

---

# 🚀 10. SMTP Service Failure (Real Experience)

## 🧩 Situation
Internal SMTP relay service failed email delivery.

## ⚙️ Action
- Checked Postfix logs  
- Found configuration mismatch in relay rules  
- Fixed config using Ansible + Terraform pipeline  

## ✅ Result
Email delivery restored across organization.

---

# 🧠 How to Use in Interviews

## 🎤 3-Step Answer Format

1. What happened  
2. What I did  
3. Result  

---

## 💡 Example

> There was a production issue where service stopped responding. I checked logs, identified config mismatch, and rolled back deployment. Service was restored within 10 minutes.

---

# ⚡ Pro Tips

- Keep answers short (2–3 minutes max)  
- Speak in structured format  
- Don’t over-explain  
- Confidence > complexity  

---

# 🚀 Outcome

Using this library you will:

✔ Avoid blank answers  
✔ Sound structured & senior  
✔ Handle DevOps incident questions confidently  
