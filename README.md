## 📄 **Assignment 2**

---

### ✅ **1.Secure Runtime Environment for Your Web Application**

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

### ✅ **2.Provisioning & Deployment Step-by-Step**

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
  ![Test Diagram](https://github.com/user-attachments/assets/f5a3aa73-deae-460a-b2f1-52f064875056)
- Topology of Azure Virtual Infrastructure:
  ![Screenshot 2025-04-08 130423](https://github.com/user-attachments/assets/20c66651-a055-485d-973f-3452a55d2ec7)
  ![Screenshot 2025-04-08 125821](https://github.com/user-attachments/assets/a8cbc2e3-6098-42b1-9149-2fa001dbe17b)
- NSG och ASG Rules
  ![Screenshot 2025-04-08 125955](https://github.com/user-attachments/assets/f3a740d4-804a-4987-bf7a-90621278b075)
- GitHub Actions workflows Screenshot:
  ![Screenshot 2025-04-08 130116](https://github.com/user-attachments/assets/1503d1d8-e5f8-45fb-ac34-083ee54390fe)
- Screenshots of Webpage from Reverseproxy
  ![Screenshot 2025-04-08 130557](https://github.com/user-attachments/assets/cea220e3-9bf4-499a-b948-d2d2529b70e5)
  ![Screenshot 2025-04-08 130650](https://github.com/user-attachments/assets/3b568bda-8d26-4048-ac40-20856ff08c9b)
- Code snippets
*cicd.yaml*
```
name: MyWebApp

on:
  push:
    branches:
    - "main"
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
        sudo systemctl stop MyWebApp.service        

    - name: Deploy the the application
      run: |
        sudo rm -Rf /opt/MyWebApp || true
        sudo cp -r /home/azureuser/actions-runner/_work/MyWebApp/MyWebApp/ /opt/MyWebApp        

    - name: Start the application service
      run: |
        sudo systemctl start MyWebApp.service
```
*GithubActionsDemo.service*
```
[Unit]
      Description=ASP.NET Web App running on Ubuntu

      [Service]
      WorkingDirectory=/opt/MyWebApp
      ExecStart=/usr/bin/dotnet /opt/MyWebApp/MyMvcApp.dll
      Restart=always
      RestartSec=10
      KillSignal=SIGINT
      SyslogIdentifier=MyWebApp
      User=www-data
      EnvironmentFile=/etc/MyWebApp/.env

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

### ✨ **Further Enhancements**
- Further developement can be implemented Azure key vault for storing Environment variables and Coonection string for CosmosDB & Azure Blob storage can be implemented as well. 
- In Virtual infrastructurem, Further individual subnets can be implemented for BastionhostVM , AppserverVM & ReverseproxyVM.
- I have used mostly Azure portal, Later on I will have to focus on Infrastructure as code and develop my skills on using CLI & ARM. Overall This project gave me good idea on How Secure infrastructure can be implemeneted on Azure.
