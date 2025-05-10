# configuring-ad
<p align="center">
<img src="https://i.imgur.com/pU5A58S.png" alt="Microsoft Active Directory Logo"/>
</p>

<h1>On-premises Active Directory Deployed in the Cloud (Azure)</h1>
This tutorial outlines the implementation of on-premises Active Directory within Azure Virtual Machines.<br />


<h2>Video Demonstration</h2>

- ### [YouTube: How to Deploy on-premises Active Directory within Azure Compute](https://www.youtube.com)

<h2>Environments and Technologies Used</h2>

- Microsoft Azure (Virtual Machines/Compute)
- Remote Desktop
- Active Directory Domain Services
- PowerShell

<h2>Operating Systems Used </h2>

- Windows Server 2022
- Windows 10 (21H2)

<h2>High-Level Deployment and Configuration Steps</h2>

- Setting up Active Directory 
- Deploying Active Directory
- Group Policy Management


<h2>Deployment and Configuration Steps</h2>

<p>
![image](https://github.com/user-attachments/assets/4a18cdde-3eda-4015-b466-b2814f12a65e)
![image](https://github.com/user-attachments/assets/111ba0fc-9a84-4df3-8576-e10985721604)

The DNS NIC(Network Interface Card) IP for client-1 will be static(doesn’t change) . The NIC private ip for dc-1 will be dynamic(can change) at first but will configured to static(doesn’t change). We don’t want client-1’s private ip to not change because we will configure the dc-1’s ip which will be a DNS Server to tell client-1 to use dc-1(domain controller/DNS server) private ip only.

</p>

<p>
•	Overall Steps
Setup Domain Controller in Azure
Create a Resource Group

Create a Virtual Network and Subnet
Create the Domain Controller VM (Windows Server 2022) named “DC-1”
	Username: labuser
	Password: Cyberlab123!
After VM is created, set Domain Controller’s NIC Private IP address to be static
Log into the VM and disable the Windows Firewall (for testing connectivity)

• Setup Client-1 in Azure
—
Create the Client VM (Windows 10) named “Client-1”
Username: labuser
Password: Cyberlab123!

Attach it to the same region and Virtual Network as DC-1
After VM is created, set Client-1’s DNS settings to DC-1’s Private IP address

You will need to copy dc-1’s private IP address and then click on the client-1 vm > then go to networking> network settings > click on the green network interface card icon at the top > click dns servers on the left > select custom so we can input dc-1s private ip address so client-1 can point to dc-1 server > save

From the Azure Portal, restart Client-1
Login to Client-1
Attempt to ping DC-1’s private IP address

•	Ensure the ping succeeded
From Client-1, open PowerShell and run ipconfig /all
•	The output for the DNS settings should show DC-1’s private IP Address 

Finish the lab, but do not delete the VMs in Azure. We will use them for upcoming labs.
If you are done for the day and want to save money, simply “Stop”/turn off the VMs within the Azure Portal

![image](https://github.com/user-attachments/assets/4cb22344-3990-4a75-ab37-3b1fc179184e)
![image](https://github.com/user-attachments/assets/55b86948-aae5-43c6-90da-16142cda73a2)
![image](https://github.com/user-attachments/assets/0cd74ea9-79b2-4fe3-a8d4-6165704b3c13)
![image](https://github.com/user-attachments/assets/a1684024-a363-4b00-ae71-c6190e36a4ce)
![image](https://github.com/user-attachments/assets/05c09b72-0983-4ca1-ba18-6f1d9bb41ff8)


<h2>Deploying Active Directory Steps</h2>

•	Overall Steps 

Turn on the DC-1 and Client-1 VMs in the Azure Portal if they are off.

•Part 1
Install Active Directory
-
Login to DC-1 and install Active Directory Domain Services
Promote as a DC: Setup a new forest as mydomain.com (can be anything, just remember what it is)
Restart and then log back into DC-1 as user: mydomain.com\labuser

• Create a Domain Admin user within the domain
-
In Active Directory Users and Computers (ADUC), create an Organizational Unit (OU) called “_EMPLOYEES”
Create a new OU named “_ADMINS”
Create a new employee named “Jane Doe” (same password) with the username of “jane_admin” / Cyberlab123!
Add jane_admin to the “Domain Admins” Security Group
Log out / close the connection to DC-1 and log back in as “mydomain.com\jane_admin”
User jane_admin as your admin account from now on

• Join Client-1 to your domain (mydomain.com)
- 
• From the Azure Portal, set Client-1’s DNS settings to the DC’s Private IP address (Already done)
From the Azure Portal, restart Client-1 (Already done)
Login to Client-1 as the original local admin (labuser) and join it to the domain (computer will restart)
Login to the Domain Controller and verify Client-1 shows up in ADUC
Create a new OU named “_CLIENTS” and drag Client-1 into there

• Finish the lab, but do not delete the VMs in Azure. We will use them for upcoming labs.
If you are done for the day and want to save money, simply “Stop”/turn off the VMs within the Azure Portal

Part 2

• Turn on the DC-1 and Client-1 VMs in the Azure Portal if they are off.
Setup Remote Desktop for non-administrative users on Client-1
- 
• Log into Client-1 as mydomain.com\jane_admin
Open system properties
Click “Remote Desktop”
Allow “domain users” access to remote desktop
You can now log into Client-1 as a normal, non-administrative user now
Normally you’d want to do this with Group Policy that allows you to change MANY systems at once (maybe a future lab)

•Create a bunch of additional users and attempt to log into client-1 with one of the users
-
• Login to DC-1 as jane_admin
Open PowerShell_ISE as an administrator
Create a new File and paste the contents of the script into it
Run the script and observe the accounts being created
When finished, open ADUC and observe the accounts in the appropriate OU　(_EMPLOYEES)
attempt to log into Client-1 with one of the accounts (take note of the password in the script)

• Finish the lab, but do not delete the VMs in Azure. We will use them for upcoming labs.
If you are done for the day and want to save money, simply “Stop”/turn off the VMs within the Azure Portal

![image](https://github.com/user-attachments/assets/a7b1802f-10c8-467d-8e36-b4224f104d74)
![image](https://github.com/user-attachments/assets/c123140b-a0f2-469f-8757-c1d409cb0f39)
![image](https://github.com/user-attachments/assets/d8f9fa8b-7309-4d0a-b293-9a5b8c55a98c)
![image](https://github.com/user-attachments/assets/e5eab926-8fee-46a9-8220-a1ebe579f6db)

<h2> Group Policy: Enabling/Unlocking accounts and Resetting Passwords </h2>

Turn on the DC-1 and Client-1 VMs in the Azure Portal if they are off.

•Dealing with Account Lockouts
Get logged into dc-1
Pick a random user account you created previously
Attempt to log in with it 10 times with a bad password

•Configure Group Policy to Lockout the account after 5 attempts:
How To Configure Account Lockout Threshold in Group Policy

•Attempt to log in with it 6 times with a bad password

•Observe that the account has been locked out within Active Directory
Unlock the account ( Click on user > select unlock account button)
Reset the password (Right click user> reset password button) 
Attempt to login with it

Enabling and Disabling Accounts
Disable the same account in Active Directory
Attempt to login with it, observe the error message
Re-enable the account and attempt to login with 

![image](https://github.com/user-attachments/assets/991cdf73-4696-4159-9770-8a7483f35742)
![image](https://github.com/user-attachments/assets/7888534e-d52a-4a65-8297-39efcb445230)
![image](https://github.com/user-attachments/assets/ad79d8aa-ca4c-45c2-8ff0-b06cf9b3d597)
![image](https://github.com/user-attachments/assets/ee80f61f-6660-4333-8142-23968e05ab53)

This is the end of the lab.
</p>
<br />

