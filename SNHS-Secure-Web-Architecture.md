# 🛡️ Enterprise Zero-Trust Web Architecture
**Project:** Science National Honor Society (SNHS) Web Infrastructure  
**Author:** Peter M Nguyen  
**Framework Alignment:** ISO/IEC 27001:2022  

---

## 💡 Overview
This project demonstrates the end-to-end deployment of a secure, highly available web architecture in Microsoft Azure. Moving beyond standard compute deployment, this environment is architected around **Zero Trust** principles, Defense-in-Depth, and strict alignment with ISO 27001 Annex A controls. 

The infrastructure utilizes Azure Native security services to segregate network traffic, eliminate standing administrative privileges, seamlessly manage cryptographic keys, and block Layer 7 application attacks natively at the edge.

---

## 🏗️ Architecture Stack & ISO Mapping

| Component | Security Function | OSI Layer | ISO 27001 Control |
| :--- | :--- | :--- | :--- |
| **Azure App Gateway (WAF V2)** | Blocks OWASP Top 10 (SQLi, XSS) | Layer 7 | **8.28** (Secure Coding/App Security) |
| **Azure Key Vault & Managed Identity** | Secretless SSL/TLS management | Layer 6 / 7 | **8.24** (Cryptography) & **8.5** (Secure Auth) |
| **Network Security Group (NSG)** | Subnet-level traffic filtering | Layer 3 / 4 | **8.20** (Networks Security) |
| **Defender for Cloud (JIT)** | Time-bound administrative access | Layer 7 | **8.2** (Privileged Access Rights) |
| **Azure DNS Zones & Static IP** | Centralized routing and RBAC | Layer 3 / 7 | **8.22** (Segregation of Networks) |

---

## 🚀 Phase 1: Compute & Network Foundation (IaaS)
To establish a secure perimeter, the application is hosted on Infrastructure as a Service (IaaS) rather than a shared PaaS environment.
* **Virtual Network (VNet):** Deployed a dedicated VNet (`10.0.0.0/16`) to act as the master private cloud boundary.
* **Network Segmentation:** Carved out dedicated, segregated subnets for the Web Server (`10.0.1.0/24`) and the Application Gateway/WAF (`10.0.2.0/24`).
* **Compute Engine:** Provisioned an Ubuntu 22.04 LTS Virtual Machine running the Nginx web server, utilizing SSH key-pair authentication over legacy password authentication.

## 🔐 Phase 2: Layer 3/4 Security & Zero Standing Privileges
Default permissive routing was disabled to harden the environment against automated network scanning and brute-force attacks.
* **The NSG Bouncer:** Attached a Network Security Group to the Web Subnet, explicitly allowing only inbound HTTP/HTTPS traffic from the WAF subnet, and denying all internet traffic.
* **Just-In-Time (JIT) VM Access:** Eliminated 24/7 standing SSH access. Administrative management (Port 22) requires an explicit, time-bound request via Microsoft Defender for Cloud, which dynamically pokes a 2-hour hole in the NSG locked to the administrator's specific IP address.

## 🌐 Phase 3: Edge Routing & Application Firewall (WAF)
To protect the underlying Linux server from malicious code injection, the public IP was abstracted away from the compute layer.
* **Azure DNS Zones:** Migrated domain registrar DNS management natively into Azure to centralize Role-Based Access Control (RBAC) over the `snhs.org` domain routing.
* **Azure Application Gateway:** Deployed an Azure App Gateway equipped with a **Web Application Firewall (WAF V2)** in the dedicated edge subnet.
* **OWASP Prevention:** Configured the WAF in Prevention Mode to actively inspect Layer 7 HTTP payloads and automatically drop known vulnerabilities (like Cross-Site Scripting and SQL Injections) before they reach the web server.

## 🔑 Phase 4: Cryptography & Identity Management
To ensure data confidentiality in transit without introducing "Secret Zero" vulnerabilities (hardcoded credentials).
* **Azure Key Vault:** Provisioned a hardware-backed Azure Key Vault to securely store the domain's SSL/TLS certificates.
* **System-Assigned Managed Identity:** Assigned a native Entra ID identity directly to the Ubuntu Virtual Machine.
* **Secretless Handshake:** Granted the VM's Managed Identity RBAC permissions to read the Key Vault. The server retrieves its cryptographic certificates dynamically, completely eliminating the need to store passwords or private keys on the local compute drive, forcing all web traffic over encrypted HTTPS.
