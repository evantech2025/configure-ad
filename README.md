<p align="center">
<img src="https://i.imgur.com/pU5A58S.png" height="40%" width="70%"alt="Microsoft Active Directory Logo"/>
</p>

<h1>Configuring Active Directory (On-Premises) Within Azure</h1>
This tutorial outlines the implementation of on-premises Active Directory within Azure Virtual Machines.<br />


<h2>Environments and Technologies Used</h2>

- Microsoft Azure (Virtual Machines)
- Remote Desktop (RDP)
- Active Directory Domain Services
- PowerShell

<h2>Operating Systems Used </h2>

- Windows Server 2022
- Windows 10 (22H2)

<h2>High-Level Deployment and Configuration Steps</h2>

- Create Resources
- Ensure Connectivity Between Client and Domain Controller
- Install Active Directory
- Create an Admin and Normal User Account in Active Directory
- Join Client-1 to Your Domain (mydomain.com)
- Setup Remote Desktop for Non-Administrative Users on Client-1
- Create Additional Users and Attempt to Log into Client-1 with one of the Users

<br/>
<h2>Deployment and Configuration Steps</h2>

---

### 🏗️ Setup Domain Controller in Azure

#### 1. Create a Resource Group  
Create a new resource group for managing related Azure resources.

![Step 1 - Create Resource Group](https://github.com/user-attachments/assets/d7157dde-82d2-4cc2-a4bc-dc269b6c266c)


---

#### 2. Create a Virtual Network and Subnet  
Configure a VNet and Subnet for DC-1 and Client-1 to communicate.

![Step 2 - Create VNet](https://github.com/user-attachments/assets/c8cb1fc7-9d74-41ae-b3a9-32af2989223e)

---

#### 3. Create the Domain Controller VM  
- OS: Windows Server 2022  
- Name: `DC-1`  
- Username: `labuser`  
- Password: `Cyberlab123!`

![Step 3 - Create DC VM](https://github.com/user-attachments/assets/312096ef-84b1-47e0-9d1d-fefb67909205)

---

#### 4. Set Static Private IP for DC-1  
Update the NIC settings to ensure a static private IP address.

![Step 4 - Set Static IP](https://github.com/user-attachments/assets/b705f1de-5106-4bf7-a416-3dc8936c71fc)

---

#### 5. Enable ICMPv4 on Windows Firewall  
- Log into DC-1 and open Windows Defender Firewall with Advanced Security.
- For inbound rules enable the "ICMP Echo Request (ICMPv4-In)" under Core Networking Diagnostics. Select the one under private profile.
This will allow ICMP/ping within our VNet.

![Step 5 - Enable ICMPv4](https://github.com/user-attachments/assets/23d695aa-f290-4e3d-ab21-b82cc2b22fca)

---

### 🖥️ Setup Client-1 in Azure

#### 6. Create Client-1 VM  
- OS: Windows 10  
- Name: `Client-1`  
- Username: `labuser`  
- Password: `Cyberlab123!`  
- Attach to same region and VNet as DC-1

![Step 6 - Create Client VM](https://github.com/user-attachments/assets/0bfbcc6c-5c5a-482f-b3f5-c9543c41dc85)

---

#### 7. Set Client-1 DNS to DC-1 IP  
Update the network interface DNS settings for Client-1 to point to DC-1’s static IP.

![Step 7 - Set DNS](https://github.com/user-attachments/assets/733d7b92-228b-4d1a-8738-a5bd90aacc81)

---

#### 8. Restart and Ping  
Restart Client-1 from Azure Portal and test connectivity by pinging DC-1.

![Step 8 - Ping DC](https://github.com/user-attachments/assets/2ea9b4d0-0578-4793-be19-b016003533bc)

---

#### 9. Verify DNS in ipconfig  
Run `ipconfig /all` on Client-1 to verify the DNS points to DC-1’s IP.

![Step 9 - IPConfig](https://github.com/user-attachments/assets/91105298-4fb6-4d5b-a8ab-8bd58d6163cb)

---

### 🔧 Part 1: Install Active Directory

#### 10. Install Active Directory Domain Services  
Login to DC-1, install ADDS, and promote it to a domain controller for a new forest (e.g., `mydomain.com`).

![Step 10 - Install ADDS](https://github.com/user-attachments/assets/af62bbf4-8f1a-4db9-8fba-e8f62baa2525)
![Step 10 - Install ADDS](https://github.com/user-attachments/assets/e58366a3-7af3-4af2-b7f2-1c755ca68741)
![Step 10 - Install ADDS](https://github.com/user-attachments/assets/0138abc4-b70a-40a4-9be6-54fd21e1ec1e)



---

#### 11. Restart and Re-login  
After promotion, restart and login as `mydomain.com\labuser`.

![Step 11 - Re-login](https://github.com/user-attachments/assets/2404f5ab-f8b2-4916-8f41-50e359992fa6)

---

### 👤 Create Domain Admin User

#### 12. Create OUs and Domain Admin  
Open Active Directory Users and Computers (ADUC) and create two Organizational Units (OUs).
- OU: `_EMPLOYEES`, `_ADMINS`

![Step 12 - Create Admin](https://github.com/user-attachments/assets/766a1303-2cd2-48af-b66d-0d4a4ce61b9b)
<br/>

Create a new user inside "_EMPLOYEES".
- User: `jane_admin` with password `Cyberlab123!`

![Step 12 - Create Admin](https://github.com/user-attachments/assets/cce5719f-2eff-4399-bb3c-a8d6bd1e1c07)


- Add to `Domain Admins` group

![Step 12 - Create Admin](https://github.com/user-attachments/assets/ee6caca0-5ee6-4fec-9a61-6a6e20e068d2)

---

#### 13. Login as jane_admin  
Use `mydomain.com\jane_admin` going forward.

![Step 13 - Login as Jane](images/step13_login_as_jane.png)

---

### 🔗 Join Client-1 to Domain

#### 14. Join Client-1 to Domain  
Log into Client-1 as `labuser` and join to `mydomain.com`.

![Step 14 - Join Domain](images/step14_join_domain.png)

---

#### 15. Verify in ADUC  
Ensure `Client-1` appears in Active Directory.

![Step 15 - Verify in ADUC](images/step15_verify_aduc.png)

---

#### 16. Organize Clients  
Create an OU named `_CLIENTS` and move `Client-1` into it.

![Step 16 - Organize OU](images/step16_organize_ou.png)

---

### 🔧 Part 2: Additional AD Setup

#### 17. Enable Remote Desktop for Domain Users  
On `Client-1`, allow Remote Desktop access to "Domain Users".

![Step 17 - RDP Access](images/step17_rdp_access.png)

---

### 👥 Create Additional Users

#### 18. Bulk Create Users with PowerShell  
- Login to `DC-1` as `jane_admin`  
- Open `PowerShell ISE` as admin  
- Paste and run the user creation script  
- Observe accounts in `_EMPLOYEES`

![Step 18 - Create Users Script](images/step18_user_script.png)

---

#### 19. Test Login with New User  
Log into `Client-1` with one of the newly created accounts.

![Step 19 - Test User Login](images/step19_test_login.png)

---

### ✅ Configuration Complete

You now have a functional Active Directory environment in Azure with client connectivity and remote access setup.
