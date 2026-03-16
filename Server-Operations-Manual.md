# 🖥️ Server Operations & Threat Hunting Manual
**Project:** Science National Honor Society (SNHS) Web Infrastructure  
**Author:** Peter M Nguyen  
**Environment:** Azure IaaS / Ubuntu 22.04 LTS  

---

## 💡 Overview
This manual documents the day-to-day administrative operations and active threat-hunting procedures for the SNHS web infrastructure. It outlines the reason behind the operating system selection, the strict protocols for secure server access, and the command-line techniques used to monitor for malicious activity. This was used to understand how to operate and defend a secure cloud environment. 

---

## 🏗️ 1. OS & Web Server Architecture Rationale

To optimize for performance, cost, and security, the application stack was carefully selected over traditional heavyweight alternatives:

* **Operating System:** **Ubuntu 22.04 LTS (Headless)**
  * *Why:* By deploying a headless Linux distribution, we eliminate the compute overhead and proprietary licensing costs associated with Windows Server. More importantly, lacking a graphical user interface (GUI) drastically reduces the underlying attack surface.
  * Other Operating System studied: Windows Server
* **Web Server:** **Nginx**
  * *Why:* Chosen over Apache due to its asynchronous, event-driven architecture. Nginx efficiently handles high-concurrency traffic spikes with a fraction of the memory footprint and eliminates decentralized `.htaccess` directory scanning, further hardening the server.

---

## 🔐 2. Secure Administrative Access (JIT)

To maintain strict adherence to **ISO 27001 Control 8.2 (Privileged Access Rights)**, the infrastructure enforces Zero Standing Privileges (ZSP). There is no permanent SSH access to this server.

**Administrator Login Protocol:**
1. **Initiate Request:** Administrator navigates to **Microsoft Defender for Cloud** in the Azure Portal.
2. **Request JIT Access:** Under *Workload Protections > Just-in-time VM access*, request access to `SNHS-Web-01`.
3. **Time-Bound Activation:** Toggle Port 22 (SSH), restrict access strictly to `My IP`, and open the port for a maximum of 2 hours.
4. **Connect:** Azure dynamically modifies the Network Security Group (NSG) to allow the connection. Proceed to connect using the secure `.pem` SSH key pair.
5. **Automated Revocation:** Upon timer expiration, Azure automatically deletes the NSG rule, dropping the port and restoring the server's invisibility to the public internet.

---

## 🕵️‍♂️ 3. Active Threat Hunting & Log Analysis

Even with an Azure App Gateway (WAF) and strict NSG rules, continuous monitoring of the host's authentication logs is critical for identifying potential internal or external intrusion attempts.

Instead of relying solely on heavy SIEM dashboards, administrators perform rapid, native log parsing directly from the Linux CLI using piped commands.

### Hunting for SSH Brute-Force Attacks
To parse the massive authentication log and isolate specific failed login attempts:

```bash
sudo cat /var/log/auth.log | grep "Failed password"
