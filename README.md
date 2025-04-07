## 📄 **Assignment 2**

---

### ✅ **1. Design a Secure Runtime Environment for Your Web Application**

#### 🔹 **Architecture Overview**
My web application is built in **C# using ASP.NET MVC** and deployed in a secure Azure environment. The infrastructure consists of the following components:

- A **Virtual Network (VNET)** with subnets, NSGs (Network Security Groups), and ASGs (Application Security Groups)
- A **Bastion Host (Linux VM)** for secure SSH access using private keys
- A **Reverse Proxy Server** (e.g., Nginx) that forwards HTTP traffic from port 80 to the application
- An **App Server (Linux VM)** that hosts the ASP.NET MVC application
- **GitHub + GitHub Actions** for Continuous Integration and Continuous Deployment (CI/CD)

#### 🔹 **Purpose of Each Component**

| Component        | Purpose |
|------------------|---------|
| **Bastion Host** | Provides secure SSH access (port 22) – protects the App Server from direct internet access |
| **Reverse Proxy**| Listens on port 80 and forwards traffic to the App Server on port 5000 |
| **App Server**   | Hosts and runs the ASP.NET MVC application using .NET 9 runtime |
| **NSG/ASG**      | Controls network traffic between components in the virtual network |
| **GitHub Actions**| Automates build and deployment process from GitHub repository to the server |

#### 🔹 **Security Measures Taken**

- **SSH access only via Bastion Host** (App Server has no public IP)
- **Private SSH keys** used for secure login 
- **NSG rules** allow only required ports (22, 80 externally, 5000 internally)
- App runs as a **non-root user** for safety
- Sensitive data such as **connection strings** are stored in a `.env` file on VM added manually in AppserverVM
- **CI/CD pipeline** ensures automated but controlled deployment, reducing human errors

#### 🔹 **Azure Cloud Services Used**

- **Azure Virtual Machines (Linux)**
- **Azure Virtual Network (VNET, NSG, ASG)**
- **GitHub and GitHub Actions** (for CI/CD workflow)

---

### ✅ **2. Define Boundaries and Describe Provisioning & Deployment Step-by-Step**

#### 🔹 **Scope of the Solution**
The application runs in an **isolated Azure environment**, protected by a VNET. All access is routed through the Bastion Host. Deployment is automated from GitHub to the App Server. Security is a core design principle across provisioning and deployment.

---

### 🛠️ **Step 1: Provision the Environment (Infrastructure)**

| Step | Description | Tools Used |
|------|-------------|------------|
| 1. | Create a Resouce Group *SecureRSG* and Virtual Network *SecureVnet* in RSG with default subnet | Azure Portal |
| 2. | Set up a *SecureNSG* and rules using *Bastionhost-asg* and *Reverseproxy-asg* (only allow ports 22, 80 externally and 5000 internally) | Azure Portal |
| 3. | Create Bastion Host VM | Ubuntu VM, SSH access via private key |
| 4. | Create App Server VM | Ubuntu VM, install .NET 9 runtime |
| 5. | Install Nginx on Reverse Proxy VM | Connect via Bastion, use SSH |
| 7. | Configure a `systemd` service to run the ASP.NET app | On App Server |

---

### 🔧 **Step 2: CI/CD & Deployment**

| Step | Description | Tools |
|------|-------------|-------|
| 1. | Push code to GitHub master branch | Git |
| 2. | GitHub Actions triggers workflow | GitHub |
| 3. | Build app using `dotnet publish` | GitHub runner |
| 4. | Use `scp` to transfer published app to App Server | SSH inside CI workflow |
| 5. | Restart app using `systemctl restart myapp` | SSH in CI |

---

### 🔐 **How Security Affects Provisioning and Deployment**

| Security Measure | Impact |
|------------------|--------|
| No public IP on App Server | Reduces exposure but requires routing through Bastion |
| SSH key authentication | Prevents brute-force and password-based attacks |
| Environment secrets stored securely | Protects sensitive data |
| Http through Reverse Proxy | Protects the Appserver VM from potential attacks from direct access from Internet |

---

### ✨ **Extra Suggestions**
- Include the architecture diagram you created in your report
  ![Test Diagram2](https://github.com/user-attachments/assets/9e5068a6-1d35-4c9d-8085-6a6b0af2addf)
- Topology of Azure Virtual Infrastructure:
  ![Screenshot 2025-04-07 135505](https://github.com/user-attachments/assets/2003c574-1704-4aba-b27a-abc9384bea25)
- GitHub Actions workflows:
  
- Use bullet points and tables for clarity
- Optionally include sample code snippets (e.g., `systemd` service file or `yaml` GitHub Action)

---

Would you like help generating the actual:
- GitHub Actions YAML file?
- `systemd` unit file for your app?
- Example `.env` file?
  
I can create those for you next!
