
# Secure 2-VM Network on Microsoft Azure

## 1. Project Overview

This project demonstrates the implementation of a secure two-VM network architecture in Microsoft Azure using a Windows Server VM and a private Ubuntu Linux VM.

The Windows VM is configured with a public IP address and acts as a **Jump Box (Jump Server)**. It provides a controlled entry point to access the private Linux VM.

The Linux VM is deployed without a public IP address and can only be accessed through the Windows VM using its private IP address.

Network Security Groups (NSGs) are configured to restrict inbound traffic and allow only authorized connections.

The project also demonstrates Infrastructure as Code (IaC) concepts using Terraform and aztfexport to export and manage existing Azure resources.

---

## 2. Project Objectives

The main objectives of this project are:

- Create an Azure Resource Group to organize project resources.
- Create a Virtual Network (VNet) with two separate subnets.
- Deploy a Windows VM with a public IP address.
- Configure the Windows VM as a secure Jump Box.
- Deploy a private Linux VM without a public IP address.
- Configure Network Security Groups (NSGs) to control inbound traffic.
- Allow RDP access to the Windows VM only from an authorized public IP.
- Allow SSH access to the Linux VM only from the Windows VM.
- Verify that direct access from the laptop to the Linux VM is blocked.
- Use Terraform and aztfexport to manage Azure infrastructure configurations.
- Test the connectivity and security of the deployed infrastructure.
- Delete the infrastructure after testing to avoid unnecessary Azure credit consumption.

---

## 3. Azure Network Architecture

The project uses a Virtual Network with two separate subnets.

### Network Configuration

| Component | Configuration |
|---|---|
| Resource Group | rg-secure-vm-lab |
| Virtual Network | vnet-secure-vm-lab |
| VNet Address Space | 10.0.0.0/16 |
| Windows Subnet | snet-windows |
| Windows Subnet CIDR | 10.0.1.0/24 |
| Linux Subnet | snet-linux |
| Linux Subnet CIDR | 10.0.2.0/24 |
| Windows Private IP | 10.0.1.4 |
| Linux Private IP | 10.0.2.4 |

### Network Architecture Diagram

```text
                       Laptop
                          |
                          |
                  RDP - TCP 3389
                          |
                          |
                   Public IP Address
                          |
                          |
                +-------------------+
                |   Windows VM      |
                |   Jump Box        |
                |   10.0.1.4        |
                +-------------------+
                          |
                          |
                   SSH - TCP 22
                          |
                          |
                +-------------------+
                |    Linux VM       |
                |    Private VM     |
                |    10.0.2.4       |
                |    No Public IP   |
                +-------------------+
```

### Network Flow

1. The laptop connects to the Windows VM using RDP over TCP port 3389.
2. The Windows VM acts as a Jump Box.
3. From the Windows VM, an SSH connection is established to the Linux VM over TCP port 22.
4. Direct access from the laptop to the Linux VM is blocked because the Linux VM does not have a public IP address and its NSG allows SSH only from the Windows VM.

---

## 4. Network Security Configuration

Two Network Security Groups (NSGs) were configured to control inbound traffic and secure access to both virtual machines.

### 4.1 Windows NSG

**Security Rule: Allow-RDP-From-my-IP**

| Setting | Value |
|---|---|
| Rule Name | Allow-RDP-From-my-IP |
| Protocol | TCP |
| Source | 122.172.86.19 |
| Source Port | * |
| Destination | Any |
| Destination Port | 3389 |
| Priority | 100 |
| Action | Allow |

This rule allows RDP access to the Windows VM only from my laptop's authorized public IP address.

### 4.2 Linux NSG

**Security Rule: Allow-SSH-From-Windows**

| Setting | Value |
|---|---|
| Rule Name | Allow-SSH-From-Windows |
| Protocol | TCP |
| Source | 10.0.1.4 |
| Source Port | * |
| Destination | Any |
| Destination Port | 22 |
| Priority | 100 |
| Action | Allow |

This rule allows SSH access to the Linux VM only from the Windows VM's private IP address.

### Security Benefits

- Restricts unauthorized inbound connections.
- Prevents direct internet access to the private Linux VM.
- Allows only authorized RDP and SSH traffic.
- Provides controlled access through the Windows Jump Box.
- Improves network isolation and security.

---

## 5. Virtual Machine Configuration

### 5.1 Windows Jump VM

| Setting | Configuration |
|---|---|
| VM Name | vm-windows-jump |
| Operating System | Windows Server 2022 Datacenter |
| Subnet | snet-windows |
| Private IP | 10.0.1.4 |
| Public IP | Assigned |
| Access Protocol | RDP |
| Port | TCP 3389 |

The Windows Server 2022 VM was deployed in the Windows subnet with a public IP address. It was configured as a Jump Box to provide secure access to the private Linux VM.

RDP access through TCP port 3389 was restricted to my laptop's public IP using the Windows NSG.

### 5.2 Private Linux VM

| Setting | Configuration |
|---|---|
| VM Name | vm-linux-private |
| Operating System | Ubuntu Server 24.04 LTS |
| Subnet | snet-linux |
| Private IP | 10.0.2.4 |
| Public IP | Not Assigned |
| Access Protocol | SSH |
| Port | TCP 22 |

The Ubuntu Linux VM was deployed in the Linux subnet (10.0.2.0/24) without a public IP address.

SSH access through TCP port 22 was restricted to the Windows VM's private IP address (10.0.1.4). This ensures that the Linux VM is not directly accessible from the internet and can only be accessed through the Windows Jump Box.

---

## 6. VM Connectivity Verification

Connectivity testing was performed to verify that the network configuration and NSG rules were working as expected.

### 6.1 Successful RDP Connection – Laptop to Windows VM

- Connected to the Windows VM from my laptop using Remote Desktop Protocol (RDP).
- Used the Windows VM's public IP address.
- Verified that the RDP connection was successful.

### 6.2 Successful SSH Connection – Windows VM to Linux VM

- Connected to the Windows VM using RDP.
- Opened PowerShell or Command Prompt inside the Windows VM.
- Established an SSH connection to the Linux VM using its private IP address (10.0.2.4).
- Verified successful SSH access.

### 6.3 Direct Laptop-to-Linux Connectivity Test

- Attempted to connect directly from the laptop to the Linux VM.
- Direct access was not allowed.
- Verified that the Linux VM could only be accessed through the Windows Jump Box.

---

## 7. Terraform Automation Using aztfexport

The Azure infrastructure was initially created manually through the Azure Portal.

To automate infrastructure management, I used **aztfexport**, a tool that exports existing Azure resources and generates Terraform configuration files.

### 7.1 Resource Export

Using aztfexport, I exported the existing Azure resources into Terraform configuration files.

The exported configuration included resources such as:

- Resource Group
- Virtual Network
- Subnets
- Network Security Groups
- NSG Security Rules
- Network Interfaces
- Virtual Machines
- Public IP Address
- SSH Public Key
- Bastion Host configuration

### 7.2 Terraform State Import

Initially, eight networking resource instances were imported into Terraform state:

- 1 Resource Group
- 1 Virtual Network
- 2 Subnets
- 2 Network Security Groups
- 2 NSG Security Rules

The generated Terraform configuration was reviewed and modified according to the project requirements.

### 7.3 Terraform Commands Used

Initialize Terraform:

```bash
terraform init
```

Format Terraform files:

```bash
terraform fmt
```

Validate the configuration:

```bash
terraform validate
```

Review the execution plan:

```bash
terraform plan
```

Deploy infrastructure:

```bash
terraform apply
```

Delete infrastructure after testing:

```bash
terraform destroy
```

### 7.4 Learning Outcomes

- Understanding Terraform configuration files.
- Exporting existing Azure resources using aztfexport.
- Understanding Terraform state management.
- Identifying resource dependencies.
- Reviewing Terraform execution plans.
- Modifying generated Terraform configurations.
- Troubleshooting infrastructure deployment issues.

---

## 8. Challenges Faced and Solutions

| Challenge | Solution |
|---|---|
| RDP connection issues | Verified the public IP and Windows NSG rules. |
| Identifying the laptop's public IP | Used PowerShell to retrieve the public IP address. |
| Windows VM login issues | Checked the administrator credentials and reset the password through Azure Portal when required. |
| SSH private key not found | Verified the key filename and file location. |
| Terraform subnet reference errors | Corrected subnet references in the NIC configuration. |
| Bastion deployment issue | Reviewed the VNet configuration and removed the unnecessary Bastion resource. |
| Terraform configuration conflicts | Reviewed and modified the generated Terraform code. |
| NSG access restrictions | Configured the correct source IP addresses and destination ports. |

---

## 9. Cleanup and Final Verification

I first created the Azure infrastructure using Terraform to verify that the resources were deployed successfully.

I then tested the connectivity between the Windows and Linux VMs to confirm that the network configuration and security rules were working as expected.

After successfully completing the testing, I deleted all the project infrastructure using Terraform to avoid unnecessary usage of Azure credits.

---

## 10. Technologies Used

| Technology | Purpose |
|---|---|
| Microsoft Azure | Cloud platform |
| Azure Virtual Machines | Hosting Windows and Linux VMs |
| Azure Virtual Network | Network isolation and connectivity |
| Subnets | Separating Windows and Linux workloads |
| Network Security Groups | Controlling inbound network traffic |
| Windows Server 2022 | Jump Box VM |
| Ubuntu Linux 24.04 | Private Linux VM |
| RDP | Remote access to Windows VM |
| SSH | Secure access to Linux VM |
| Terraform | Infrastructure as Code |
| aztfexport | Exporting existing Azure resources |

---

## 11. Key Learnings

Through this hands-on project, I gained practical experience in:

- Creating and configuring Azure networking resources.
- Understanding Virtual Networks and subnet segmentation.
- Deploying and managing Windows and Linux virtual machines.
- Implementing a Jump Box architecture.
- Configuring NSG rules and restricting inbound traffic.
- Understanding public and private IP addresses.
- Establishing secure RDP and SSH connections.
- Using aztfexport to generate Terraform configurations.
- Managing Terraform state and resource dependencies.
- Troubleshooting Azure networking and deployment issues.
- Cleaning up cloud infrastructure after project completion.

---

## 12. Project Demonstration

**Project Demonstration Video:**  
[Watch the Complete Project Demonstration](https://drive.google.com/file/d/1hU5i-Y37JRfG7rLjeSzK7pL3PZJ5QUz6/view?usp=drive_link)

---

## 13. Conclusion

This project provided hands-on experience in building a secure Azure network using two virtual machines. It demonstrated how a Jump Box, separate subnets, and NSG rules can improve network security by controlling access to private resources.

I also gained practical experience with Terraform and aztfexport, including exporting existing Azure resources, managing infrastructure configurations, testing connectivity, and troubleshooting deployment issues.

Overall, this project strengthened my understanding of Azure networking, cloud security, and Infrastructure as Code (IaC).
