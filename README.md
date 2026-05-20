# configuring-ad
<p align="center">
<img src="https://i.imgur.com/pU5A58S.png" alt="Microsoft Active Directory Logo"/>
</p>

## Lab 1 — Active Directory
Windows Server 2025 · Azure · Identity & Access Management

<img width="1440" height="890" alt="image" src="https://github.com/user-attachments/assets/d826ecd3-a704-4001-909c-160a0e44524f" />


## Lab Details

| Field | Value |
|---|---|
| Certification alignment | CompTIA Network+ · Security+ · Azure Administrator |
| Free tools | Windows Server 2025 Evaluation (180 days) · Azure Free Account |
| Time to complete | 3–5 hours across multiple sessions |
| Estimated cost | $0 — fully covered by free tiers and evaluation licenses |
| Career relevance | IT Support · Sysadmin · Cloud Engineer · Security Analyst |

## What you will learn

| Field | Value |
|---|---|
| Skill	| Real-world application |
| Promote a Windows Server to Domain Controller	| The first step in every enterprise Windows environment |
| Create Organizational Units (OUs)	| OUs are the folders of Active Directory — they let you apply different policies to different departments |
| Create users, groups, and group memberships	| Every access decision in an enterprise is group-based |
| Configure Group Policy Objects (GPOs)	| Enforce settings across every machine in the domain centrally |
| Deploy and join a client VM to the domain	| Connect a workstation so it becomes a managed, policy-enforced resource | 
| Verify GPO application on a client machine	| Confirm policies are actually applying to end user machines | 
| Reset passwords and manage account lifecycle	| The most frequent real-world task for IT support |


<h1>On-premises Active Directory Deployed in the Cloud (Azure)</h1>
This tutorial outlines the implementation of on-premises Active Directory within Azure Virtual Machines.<br />


<h2>Video Demonstration</h2>

- ### https://www.loom.com/share/a108f66972b841b3af00c6d4aa5e4035

<h2>Environments and Technologies Used</h2>
Before starting this lab you need two virtual machines in Azure. Both must be on the same VNet and subnet so they can communicate with each other.

| Field | Value |
|---|---|
| VM     | Role   | 
|DC-1 	 |Domain Controller | 
|Client1 |Client Workstation  |

- Microsoft Azure (Virtual Machines)
- Remote Desktop
- Active Directory Domain Services
- PowerShell

<h2>Operating Systems Used </h2>

- Windows Server 2025
- Windows 10 Enterprise (win10-22h2-ent-g2) or Windows 11 (your choice)

<h2>High-Level Deployment and Configuration Steps</h2>

- Setting up Active Directory 
- Deploying Active Directory
- Group Policy Management


<h2>Deployment and Configuration Steps</h2>

<p>
![image](https://github.com/user-attachments/assets/4a18cdde-3eda-4015-b466-b2814f12a65e)
![image](https://github.com/user-attachments/assets/111ba0fc-9a84-4df3-8576-e10985721604)


</p>
•	Overall Steps
-Setup Domain Controller in Azure
-Create a Resource Group
-Create a Virtual Network and Subnet
-Create the Domain Controller VM (Windows Server 2025) named “DC-1”
-Create the Client VM (Windows 10 Enterprise) named “Client1”

Sign in to portal.azure.com → Virtual Machines → Create

| Field | Value |
|---|---|
|Setting  | Value |
|Name	|  DC-1 |
|Region	|  East US | 
|Image	| Windows Server 2025 Datacenter — Gen2 |
|Size	|   Standard_D2s_v3 (2 vCPU, 8GB RAM) |
|Username |  labuser (or whatever you want) |
|Password | strong password — write it down |
|Public inbound ports |	Allow RDP (3389) |
|OS disk   | Standard SSD |

Click Review + Create → Create and wait for deployment to complete.

VM 2 — Client Machine (client01)
Go to Virtual Machines → Create again.

| Field | Value |
|---|---|
|Setting |	Value |
|Name	| Client1 |
|Region	 |East US — must match DC |
|Image	| Windows 10 Enterprise (win10-22h2-ent-g2)  |
|Size	|Standard_D2s_v3 |
|Username	|labuser |
|Password |	same password as DC |
| Public inbound ports| 	Allow RDP (3389) — must be selected |
| Virtual network	| select the same VNet as vm-ADDS-lab |
| Subnet |	select the same subnet as vm-ADDS-lab |

Click Review + Create → Create and wait for deployment to complete.

After Client1 deploys — set DNS on the NIC before moving on.

## You will need to copy DC-1’s private IP address

Go to Virtual Machines → Client1 → Networking → Network settings
Click on the green network interface card icon at the top
In the left menu click Settings → DNS Servers
Switch from Inherit from virtual network to Custom so client-1 can point to dc-1 server 
Enter the DC's private IP address
Click Save
Go back to Virtual Machines → client01 → Restart

Note: The DNS NIC(Network Interface Card) IP for Client-1 will be static(doesn’t change) . The NIC private ip for dc-1 will be dynamic(can change) at first but will configured to static(doesn’t change). We don’t want client-1’s private ip to not change because we will configure the dc-1’s ip which will be a DNS Server to tell client-1 to use dc-1(domain controller/DNS server) private ip only.

Attach it to the same region and Virtual Network as DC-1
After VM is created, set Client-1’s DNS settings to DC-1’s Private IP address


<p>
	
- From the Azure Portal, restart Client-1
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

--Turn on the DC-1 and Client-1 VMs in the Azure Portal if they are off.

## Enable clipboard for RDP on both VMs
-Open Remote Desktop on your local machine
-Enter the VM's public IP
-Click Show Options → Local Resources tab
-Make sure Clipboard is checked
-Click Connect


•Part 1
Step 2 - Install Active Directory Domain Services
-
## RDP into DC-1 using its public IP address. Open Server Manager — it opens automatically on login.

Note : What is a Domain Controller? A Domain Controller (DC) is a server that runs Active Directory. It is the brain of the entire identity system. When a user logs in anywhere on the domain their credentials are checked against the Domain Controller. There is usually more than one in an enterprise for redundancy but we are building one here.

- In Server Manager: Click Manage → Then Add Roles and Features → Next → Server Roles → check Active Directory Domain Services → Add Features → Install

- Wait for installation to complete — takes 2–3 minutes. When complete click Close — do not restart yet.

- Or run this in PowerShell on the DC:

##  Install-WindowsFeature -Name AD-Domain-Services -IncludeManagementTools

<img width="1847" height="867" alt="image" src="https://github.com/user-attachments/assets/e3b484ab-6e24-494a-bde4-1597dafa2b1b" />

- Also install Group Policy Management Console (GPMC) now

- Step 5 requires the Group Policy Management Console. Install it now so it is ready when you need it.

-Heres the Powershell script 
## Install-WindowsFeature -Name GPMC
<img width="1872" height="940" alt="image" src="https://github.com/user-attachments/assets/7a7cc26d-6426-4f0f-b3d9-7aa63d8fc949" />

## Step 3 — Promote the Server to a Domain Controller

-Login to DC-1 and install Active Directory Domain Services
-Promote as a DC: Setup a new forest as mydomain.com (can be anything, just remember what it is)

Note: What is a Forest and Domain? A Forest is the top-level container of your entire Active Directory structure. A Domain is a boundary inside the forest with a name — ours is lab.local. Most small-to-medium organizations have one domain inside one forest.

-In Server Manager click the yellow warning flag at the top right
-Click Promote this server to a domain controller
-Select Add a new forest
-Set Root domain name to: lab.local
-Click Next — set a DSRM password and write it down
-Click through DNS Options and NetBIOS pages — accept the defaults
-Click Install — the server will automatically restart when complete

-Or promote via PowerShell:
## Import-Module ADDSDeployment
Install-ADDSForest `
  -DomainName 'mydomain.com' `
  -DomainNetBiosName 'MYDOMAIN' `
  -InstallDns:$true `
  -SafeModeAdministratorPassword (ConvertTo-SecureString 'YourDSRMPassword!' -AsPlainText -Force) `
  -Force:$true

After that, restart RDP back into DC-1. You are now logged into a Domain Controller.
You can also log back into DC-1 as the user: mydomain.com\labuser

Step 4 — Build the Organizational Structure
Open Active Directory Users and Computers (ADUC) from the Tools menu in Server Manager.
• Create a Domain Admin user within the domain
- 
Note: What is an Organizational Unit (OU)? An OU is a folder inside Active Directory. You use OUs to organize users and computers by department. The real power is that you can link a Group Policy to an OU — every user or computer inside automatically gets the policy applied.

-Create Organizational Units w/ Powershell Script

New-ADOrganizationalUnit -Name "IT"            -Path "DC=mydomain,DC=com"
New-ADOrganizationalUnit -Name "Finance"       -Path "DC=mydomain,DC=com"
New-ADOrganizationalUnit -Name "HR"            -Path "DC=mydomain,DC=com"
New-ADOrganizationalUnit -Name "Sales"         -Path "DC=mydomain,DC=com"
New-ADOrganizationalUnit -Name "Computers"     -Path "DC=mydomain,DC=com"

-Create Security Groups w/ Powershell Script

New-ADGroup -Name "IT_Admins"     -GroupScope Global -GroupCategory Security -Path "OU=IT,DC=mydomain,DC=com"
New-ADGroup -Name "Finance_Users" -GroupScope Global -GroupCategory Security -Path "OU=Finance,DC=mydomain,DC=com"
New-ADGroup -Name "HR_Users"      -GroupScope Global -GroupCategory Security -Path "OU=HR,DC=mydomain,DC=com"
New-ADGroup -Name "Sales_Users"   -GroupScope Global -GroupCategory Security -Path "OU=Sales,DC=mydomain,DC=com"

## Create Multiple User Accounts
Important: Run the entire block below at once — not line by line. The $password variable must be defined before the New-ADUser commands or PowerShell will fail.

# Run this entire script together 
$password = ConvertTo-SecureString "Welcome@2026!" -AsPlainText -Force

#Creates 4 users
New-ADUser -Name "alice.chen" -GivenName "Alice" -Surname "Chen" `
  -SamAccountName "alice.chen" -UserPrincipalName "alice.chen@mydomain.com" `
  -Path "OU=IT,DC=mydomain,DC=com" -AccountPassword $password -Enabled $true

New-ADUser -Name "bob.patel" -GivenName "Bob" -Surname "Patel" `
  -SamAccountName "bob.patel" -UserPrincipalName "bob.patel@mydomain.com" `
  -Path "OU=Finance,DC=mydomain,DC=com" -AccountPassword $password -Enabled $true

New-ADUser -Name "carol.jones" -GivenName "Carol" -Surname "Jones" `
  -SamAccountName "carol.jones" -UserPrincipalName "carol.mydomain.com" `
  -Path "OU=HR,DC=mydomain,DC=com" -AccountPassword $password -Enabled $true

New-ADUser -Name "david.smith" -GivenName "David" -Surname "Smith" `
  -SamAccountName "david.smith" -UserPrincipalName "david.smith@mydomain.com" `
  -Path "OU=Sales,DC=mydomain,DC=com" -AccountPassword $password -Enabled $true

#Adds each user to their department
Add-ADGroupMember -Identity "IT_Admins"     -Members "alice.chen"
Add-ADGroupMember -Identity "Finance_Users" -Members "bob.patel"
Add-ADGroupMember -Identity "HR_Users"      -Members "carol.jones"
Add-ADGroupMember -Identity "Sales_Users"   -Members "david.smith"

## Step 5 — Configure Group Policy
- Open Group Policy Management from the Tools menu in Server Manager on the DC.

Note: What is a Group Policy Object (GPO)? A GPO is a collection of settings applied automatically to every user or computer inside an OU. Password complexity requirements, screen lock timers, USB restrictions — all enforced from a single GPO across every machine without touching each one individually.

- Expand Forest: mydomain.com → Domains → mydomain.com
- Right-click the IT OU → Create a GPO in this domain and link it here
- Name it: IT Security Policy
- Right-click the new GPO → Edit

## Configure all four settings below
| Policy Path | Setting| Value | Explanation |
|---|---|---|---|
| Computer Config → Windows Settings → Security → Account Policies → Password Policy| Minimum password length|12 | Enforces strong passwords 
| Computer Config → Windows Settings → Security → Account Policies → Password Policy| Password must meet complexity requirments |Enabled | Enabled	Requires upper, lower, number, and symbol| 
| Computer Config → Windows Settings → Security → Local Policies → Security Options|Interactive logon: Machine inactivity limit |900 seconds| 	Auto-locks screen after 15 minutes| 
| Computer Config → Administrative Templates → System → Removable Storge Access |All removable storage classes: Deny all access| Enabled | Blocks USB drives

## Step 6 — Join client01 to the Domain and Test GPO
Note: This step uses the Client1 VM you created in Step 1. This is how you verify that your GPO is actually working — without a client machine you can't confirm that the  policies you configured were applyied correctly.

- Part A — Join client01 to the domain
RDP into client01 using its public IP address. Login with:

- Username: .\labuser
- Password: your password
- Run this command inside client01:

## Add-Computer -DomainName "mydomain.com" -Credential (Get-Credential) -Restart -Force
A credentials popup will appear. Enter:

Username: MYDOMAIN\your-dc-admin-account
Password: your DC password
The VM will restart automatically confirming the domain join worked.

- Part B — Move client01 to the IT OU
- After the restart verify the domain join on client01:

## (Get-WmiObject Win32_ComputerSystem).Domain
Expected output:
mydomain.com

-On the DC move Client1 to the IT OU so the IT Security Policy GPO applies to it:

# First find where client01 landed
## Get-ADComputer -Filter {Name -eq "client01"} | Select-Object Name, DistinguishedName

# Move it to the IT OU

## Move-ADObject `
  -Identity "CN=Client1,CN=Computers,DC=mydomain,DC=com" `
  -TargetPath "OU=IT,DC=mydomain,DC=com"

# Verify
## Get-ADComputer -Identity "Client1" | Select-Object Name, DistinguishedName
Expected output:

CN=Client1,OU=IT,DC=mydomain,DC=com

- Part C — Test GPO application on Client1
-RDP back into client01 and run:

## gpupdate /force
- Then verify what GPOs are applying:

## gpresult /r

- Look for IT Security Policy under Computer Settings → Applied Group Policy Objects.

- Part D — Verify the inactivity lock
Leave client01 completely idle for 15 minutes. The screen will lock automatically proving the inactivity policy is working.

## Step 7 — Common Help Desk Tasks
Run all of these on the DC.

- Reset a password

## Set-ADAccountPassword -Identity "bob.patel" -Reset `
##  -NewPassword (ConvertTo-SecureString "NewPass@2026!" -AsPlainText -Force)
## Set-ADUser -Identity "bob.patel" -ChangePasswordAtLogon $true

- Unlock a locked account
## Unlock-ADAccount -Identity "carol.jones"

-Disable an account
## Disable-ADAccount -Identity "david.smith"
## Search-ADAccount -AccountDisabled | Select-Object Name, SamAccountName

- Audit inactive accounts
## $cutoff = (Get-Date).AddDays(-90)
## Get-ADUser -Filter {LastLogonDate -lt $cutoff -and Enabled -eq $true}  -Properties LastLogonDate | Select-Object Name, LastLogonDate
## Get-ADPrincipalGroupMembership -Identity "alice.chen" | Select-Object Name

## Troubleshooting
| Field | Value |
|---|---|
|Problem| Solution |
| PowerShell prompts for Name: when creating users |	Run the entire script block at once — the $password line must come first |
| Cannot copy and paste into VM	 | Open RDP → Show Options → Local Resources → check Clipboard |
| Promotion fails: DNS conflict	| Set the NIC's preferred DNS to 127.0.0.1 before promoting | 
| Cannot RDP after domain join	| Login as LAB\your-admin-account not just the local username | 
|Domain join fails: username or password incorrect	| Run Get-ADGroupMember -Identity "Domain Admins" on the DC to find the exact admin account name | 
| Client1 created as Linux VM	| Delete and recreate — make sure Image is set to Windows Server 2025 Datacenter Gen2 |
| GPO not applying on client01	 | Run gpupdate /force then gpresult /r to see what policies are applied |
| Move-ADObject fails for Client1	| Run Get-ADComputer -Filter {Name -eq "Client1"} first to find the exact DistinguishedName |
| Screen disconnects instead of locking over RDP | 	This is normal RDP behavior — use rundll32.exe user32.dll,LockWorkStation to force a visible lock screen | 
| User cannot log in after creation | 	Confirm account is Enabled and password meets complexity requirements |
| AD Users and Computers not showing	|Run dsa.msc from the Run dialog |

## Bonus Material
## Create BULK or Many User Accounts
-Alternatively we can create bulk users and simulate password resets,account unlocks, etc. with creating our new admin account and users. 
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
-Open PowerShell_ISE as an administrator
-Create a new File and paste the contents of the script into it
-The script is below(Run the whole script like this)

 # ----- Edit these Variables for your own Use Case ----- #
$PASSWORD_FOR_USERS   = "Password1"
$NUMBER_OF_ACCOUNTS_TO_CREATE = 500
# ------------------------------------------------------ #

Function generate-random-name() {
    $consonants = @('b','c','d','f','g','h','j','k','l','m','n','p','q','r','s','t','v','w','x','z')
    $vowels = @('a','e','i','o','u','y')
    $nameLength = Get-Random -Minimum 3 -Maximum 7
    $count = 0
    $name = ""

    while ($count -lt $nameLength) {
        if ($($count % 2) -eq 0) {
            $name += $consonants[$(Get-Random -Minimum 0 -Maximum $($consonants.Count - 1))]
        }
        else {
            $name += $vowels[$(Get-Random -Minimum 0 -Maximum $($vowels.Count - 1))]
        }
        $count++
    }

    return $name

}

$count = 1
while ($count -lt $NUMBER_OF_ACCOUNTS_TO_CREATE) {
    $fisrtName = generate-random-name
    $lastName = generate-random-name
    $username = $firstName + '.' + $lastName
    $password = ConvertTo-SecureString $PASSWORD_FOR_USERS -AsPlainText -Force

    Write-Host "Creating user: $($username)" -BackgroundColor Black -ForegroundColor Cyan
    
    New-AdUser -AccountPassword $password `
               -GivenName $firstName `
               -Surname $lastName `
               -DisplayName $username `
               -Name $username `
               -EmployeeID $username `
               -PasswordNeverExpires $true `
               -Path "ou=_EMPLOYEES,$(([ADSI]`"").distinguishedName)" `
               -Enabled $true
    $count++

-Run the script and observe the accounts being created
-When finished, open ADUC and observe the accounts in the appropriate OU　(_EMPLOYEES)
-Attempt to log into Client-1 with one of the accounts (take note of the password in the script)

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

-Enabling and Disabling Accounts
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

