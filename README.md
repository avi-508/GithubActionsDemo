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
| 3. | Build and deloy app to VM | using GitHub self-hosted runner |

---

### 🔐 **How Security Affects Provisioning and Deployment**

| Security Measure | Impact |
|------------------|--------|
| No public IP on App Server | Reduces exposure but requires routing through Bastion |
| SSH key authentication | Prevents brute-force and password-based attacks |
| Environment File stored securely | Protects sensitive data |
| Http through Reverse Proxy | Protects the Appserver VM from potential attacks from direct access from Internet |

---

### ✨ **Additional Screenshots**
- Architecture diagram
  ![Test Diagram2](https://github.com/user-attachments/assets/9e5068a6-1d35-4c9d-8085-6a6b0af2addf)
- Topology of Azure Virtual Infrastructure:
  ![Screenshot 2025-04-07 135505](https://github.com/user-attachments/assets/2003c574-1704-4aba-b27a-abc9384bea25)
  ![Screenshot 2025-04-07 141609](https://github.com/user-attachments/assets/38d74874-6963-4078-a95e-41dab0ce1cb6)
- NSG och ASG Rules
  ![Screenshot 2025-04-07 141229](https://github.com/user-attachments/assets/8b423418-7567-4692-84d0-d9b80a59aabd)
- GitHub Actions workflows Screenshot:
  ![Screenshot 2025-04-07 140122](https://github.com/user-attachments/assets/f352b511-501a-45c5-a635-f1b9fdebe770)
- Code snippets
*cicd.yaml*
```
name: GithubActionsDemo

on:
  push:
    branches:
    - "master"
  workflow_dispatch:

jobs:
  build:
    runs-on: ubuntu-latest
    steps:

    - name: Install .NET SDK
      uses: actions/setup-dotnet@v4
      with:
        dotnet-version: '9.0.x'

    - name: Check out this repo
      uses: actions/checkout@v4

    - name: Restore dependencies (install Nuget packages)
      run: dotnet restore

    - name: Build and publish the app
      run: |
        dotnet build --no-restore
        dotnet publish -c Release -o ./publish        

    - name: Upload app artifacts to Github
      uses: actions/upload-artifact@v4
      with:
        name: app-artifacts
        path: ./publish
        
  deploy:
    runs-on: self-hosted
    needs: build

    steps:
    - name: Download the artifacts from Github (from the build job)
      uses: actions/download-artifact@v4
      with:
        name: app-artifacts

    - name: Stop the application service
      run: |
        sudo systemctl stop GithubActionsDemo.service        

    - name: Deploy the the application
      run: |
        sudo rm -Rf /opt/GithubActionsDemo || true
        sudo cp -r /home/azureuser/actions-runner/_work/GithubActionsDemo/GithubActionsDemo/ /opt/GithubActionsDemo        

    - name: Start the application service
      run: |
        sudo systemctl start GithubActionsDemo.service        
```
*GithubActionsDemo.service*
```
[Unit]
Description=ASP.NET Web App running on Ubuntu
[Service]
WorkingDirectory=/opt/GithubActionsDemo
ExecStart=/usr/bin/dotnet /opt/GithubActionsDemo/GithubActionsDemo.dll
Restart=always
RestartSec=10
KillSignal=SIGINT
SyslogIdentifier=GithubActionsDemo
User=www-data
EnvironmentFile=/etc/GithubActionsDemo/.env

[Install]
WantedBy=multi-user.target
```
*.env*
```
Environment=ASPNETCORE_ENVIRONMENT=Production
Environment=DOTNET_PRINT_TELEMETRY_MESSAGE=false
Environment="ASPNETCORE_URLS=http://*:5000"
```
---
