Absolutely! Here's the **same documentation in English**, clearly structured and aligned with your assignment requirements.

---

## 📄 **Assignment Documentation – Basic Cloud Application (Azure + ASP.NET MVC)**

---

### ✅ **1. Design a Secure Runtime Environment for Your Web Application**

#### 🔹 **Architecture Overview**
My web application is built in **C# using ASP.NET MVC** and deployed in a secure Azure environment. The infrastructure consists of the following components:

- A **Virtual Network (VNET)** with subnets, NSGs (Network Security Groups), and ASGs (Application Security Groups)
- A **Bastion Host (Linux VM)** for secure SSH access using private keys
- A **Reverse Proxy Server** (e.g., Nginx) that forwards HTTP traffic from port 80 to the application
- An **App Server (Linux VM)** that hosts the ASP.NET MVC application
- **Cosmos DB** as the NoSQL document database
- **Azure Blob Storage** for storing media files and documents
- **GitHub + GitHub Actions** for Continuous Integration and Continuous Deployment (CI/CD)

#### 🔹 **Purpose of Each Component**

| Component        | Purpose |
|------------------|---------|
| **Bastion Host** | Provides secure SSH access (port 22) – protects the App Server from direct internet access |
| **Reverse Proxy**| Listens on port 80 and forwards traffic to the App Server on port 5000 |
| **App Server**   | Hosts and runs the ASP.NET MVC application using .NET 9 runtime |
| **NSG/ASG**      | Controls network traffic between components in the virtual network |
| **Cosmos DB**    | Stores structured or semi-structured app data (e.g., user profiles, product listings) |
| **Blob Storage** | Used for uploading and retrieving files (e.g., user images, documents) |
| **GitHub Actions**| Automates build and deployment process from GitHub repository to the server |

#### 🔹 **Security Measures Taken**

- **SSH access only via Bastion Host** (App Server has no public IP)
- **Private SSH keys** used for secure login
- **NSG rules** allow only required ports (22, 80 externally, 5000 internally)
- App runs as a **non-root user** for safety
- Sensitive data such as **connection strings** are stored in a `.env` file and secured using **GitHub Secrets**
- **HTTPS** (can be configured in Nginx using Let’s Encrypt or a self-signed certificate)
- **CI/CD pipeline** ensures automated but controlled deployment, reducing human errors

#### 🔹 **Azure Cloud Services Used**

- **Azure Virtual Machines (Linux)**
- **Azure Virtual Network (VNET, NSG, ASG)**
- **Azure Cosmos DB**
- **Azure Blob Storage**
- **GitHub and GitHub Actions** (for CI/CD workflow)

---

### ✅ **2. Define Boundaries and Describe Provisioning & Deployment Step-by-Step**

#### 🔹 **Scope of the Solution**
The application runs in an **isolated Azure environment**, protected by a VNET. All access is routed through the Bastion Host. Deployment is automated from GitHub to the App Server. Security is a core design principle across provisioning and deployment.

---

### 🛠️ **Step 1: Provision the Environment (Infrastructure)**

| Step | Description | Tools Used |
|------|-------------|------------|
| 1. | Create a VNET with two subnets | Azure Portal or Azure CLI |
| 2. | Set up NSG rules (only allow ports 22, 80 externally and 5000 internally) | Azure CLI |
| 3. | Create Bastion Host VM | Ubuntu VM, SSH access via private key |
| 4. | Create App Server VM | Ubuntu VM, install .NET 9 runtime |
| 5. | Install Nginx on Reverse Proxy VM | Connect via Bastion, use SSH |
| 6. | Create Cosmos DB and Blob Storage accounts | Azure Portal |
| 7. | Configure a `systemd` service to run the ASP.NET app | On App Server |

---

### 🔧 **Step 2: CI/CD & Deployment**

| Step | Description | Tools |
|------|-------------|-------|
| 1. | Push code to GitHub main branch | Git |
| 2. | GitHub Actions triggers workflow | GitHub |
| 3. | Build app using `dotnet publish` | GitHub runner |
| 4. | Use `scp` to transfer published app to App Server | SSH inside CI workflow |
| 5. | Restart app using `systemctl restart myapp` | SSH in CI |

✅ Sensitive credentials like SSH keys and connection strings are stored securely in **GitHub Secrets**

---

### 🔐 **How Security Affects Provisioning and Deployment**

| Security Measure | Impact |
|------------------|--------|
| No public IP on App Server | Reduces exposure but requires routing through Bastion |
| SSH key authentication | Prevents brute-force and password-based attacks |
| Environment secrets stored securely | Protects sensitive data |
| HTTPS through Reverse Proxy | Secures communication between client and server |
| Network segmentation with NSG | Limits potential attack surface |

---

### ✨ **Extra Suggestions**
- Include the architecture diagram you created in your report
- Add screenshots of:
  - Your Azure resources
  - GitHub Actions workflows
- Use bullet points and tables for clarity
- Optionally include sample code snippets (e.g., `systemd` service file or `yaml` GitHub Action)

---

Would you like help generating the actual:
- GitHub Actions YAML file?
- `systemd` unit file for your app?
- Example `.env` file?
  
I can create those for you next!
