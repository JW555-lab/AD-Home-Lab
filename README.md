# AD-Home-Lab
Active-Directory, Windows-Server, PowerShell, VirtualBox, Cybersecurity

# Building An Active Directory Domain Controller From Scratch

In this tutorial, I build a Windows Server 2022 Active Directory environment from the ground up

## Environments and Technologies Used
- Oracle VirtualBox
- Windows Server 2022 Standard Evaluation
- Windows 11 Enterprise Evaluation
- Powershell
- Active Directory Domain Services
- SConfig

## Operating Systems Used
- Windows Server 2022
- Windows 11 Enterprise

## Actions and Observations

Welcome to my walkthrough on building an Active Directory domain from the ground up. Active Directory is Microsoft's directory service - it's what lets a business centrally manage users, computers, and permissions across an entire network from one place, rather than configuring each machine individually. To build one, you need at least one server acting as a domain controller: the machine that holds the directory database and handles authentication for everything joined to it.

1. To start, I created two virtual machines in VirtualBox: one running Windows Server 2022 (DC1), which would become the domain controller, and one running Windows 11 (Client01), which would later join the domain. Both machines sit on an isolated internal network in VirtualBox so they can talk to each other without touching my host network 

<img width="966" height="750" alt="02 VM Configuration" src="https://github.com/user-attachments/assets/a80aa27c-881e-49b7-99c3-1b91e8abc4ee" />

2. With Windows Server installed, I had both VMs running side by side

   <img width="1919" height="977" alt="03 Both Vms Running" src="https://github.com/user-attachments/assets/972901c4-a91f-436d-9157-1819db5b1408" />

3. Since this build uses Server Core, there's no desktop GUI, everything from here is done through SConfig, Microsoft's text menu configuration tool for headless servers, and PowerShell. First, I set the computer name to DC1 using SConfig's numbered menu.

   <img width="1919" height="982" alt="04 Computer Name Set" src="https://github.com/user-attachments/assets/35bff801-1797-4dae-97f8-d9925c9647dc" />

4. Next, I needed a static IP. A domain controller needs a predictable, unchanging address on the network since every other machine will point to it for authentication and DNS, if its IP shifted, the whole domain would break. SConfig's own static IP option didn't reliably apply, so I set it directly in PowerShell instead.
   ( New-NetIPAddress -InterfaceAlias " Ethernet" -IPAddress 192.168.10.1 -Prefixlength 24 )

  
5. Confirmed with Ipconfig that the address took.

<img width="1919" height="914" alt="05 Static Ip Confirmed" src="https://github.com/user-attachments/assets/0777cd5e-bd07-4728-a828-472186fa8c0d" />

6. With networking sorted, I installed the Active Directory Domain Services role. The Windows Server feature that actually provides directory services. ( Install-WindowsFeature AD-Domain-Services -IncludeManagmentTools )

   <img width="1919" height="881" alt="06 AddsForest Installed" src="https://github.com/user-attachments/assets/29392355-f636-4eaf-8378-4d9175f9b90b" />

7. Then I promoted the server to a domain controller, which created a new forest. The top level container for one or more domains and the root of the entire directory structure.

8. The server rebooted automatically once the promotion finished. Logging back in showed the domain now attached to the login prompt ( JOELLAB\Administrator ), and running Get-ADDomain confirmed the domain's full details, SID, forest, DNS root, and NetBIOS name were all correctly set.

     <img width="1919" height="881" alt="07 Domain Details" src="https://github.com/user-attachments/assets/13c24255-8184-4f02-b3e1-f4e9e1f0ea5a" />

9. With the domain live, I built out its structure. Organizational Units are containers used to organize users, computers, and groups, typically by department, so policies and permissions can be applied to a whole group instead of one account at a time. I created three to represent departments.

<img width="1919" height="915" alt="08 OUs Created" src="https://github.com/user-attachments/assets/5b1ca204-bbb2-4526-b30b-55696bc719a2" />

10. Then I populated them with user accounts.

<img width="1918" height="872" alt="09 Users Created" src="https://github.com/user-attachments/assets/9bbb628a-2fcf-46f2-b8ef-ce9a930732fb" />

10. Finally, I created security groups and assigned users to them. Security groups let you assign permissions once at the group level, like IT-Admins can access the file server, instead of managing access per individual user:

<img width="1919" height="923" alt="10 Groups and Membership" src="https://github.com/user-attachments/assets/5501461d-8fce-4120-a794-ad30eba30697" />

Challenges & Troubleshooting

The biggest obstacle I ran into was installing Windows Server. I kept getting a recurring license terms error during setup, which I diagnosed methodically rather than just redownloading and hoping, verifying the ISO's integrity first, then isolating the problem to VirtualBox's unattended install feature rather than the install media itself.

SConfig's static IP option also silently failed to apply more than once. I caught this by checking the actual state with ipconfig rather than trusting the menu's confirmation, then resolved it by setting the address directly through PowerShell.

Next Steps
- Join Client01 to the domain
- Create and link group policy objects
- Implement baseline hardening (Password policy, disable Guest account, rename default Administrator)

