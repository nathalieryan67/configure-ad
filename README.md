<p align="center">
<img src="https://i.imgur.com/pU5A58S.png" alt="Microsoft Active Directory Logo"/>
</p>

<h1>On-premises Active Directory Deployed in the Cloud (Azure)</h1>
This tutorial outlines the implementation of on-premises Active Directory within Azure Virtual Machines.<br />


<h2>Environments and Technologies Used</h2>

- Microsoft Azure (Virtual Machines/Compute)
- Remote Desktop
- Active Directory Domain Services
- PowerShell

<h2>Operating Systems Used </h2>

- Windows Server 2022
- Windows 10 (21H2)

<h2>High-Level Deployment and Configuration Steps</h2>

- Create two virtual machines (Client-1 and DC-1)
- Install Active Directory on the DC-1 VM
- Create a domain admin user within the domain
- Join the Client-1 VM to the domain of the DC-1 VM
- Setup remote desktop for non-administrative users on the Client-1 VM
- Create many additional users & log into the Client-1 VM with one of the new users

<h2>Deployment and Configuration Steps</h2>

<p>
<img src="https://i.imgur.com/6Cr3DKk.png" height="80%" width="80%"/>
</p>
<p>
First, we create two virtual machines (VMs): One, named DC-1, runs on Windows Server 2022, and the other, named Client-1, runs on Windows 10. Both VMs are connected to the same virtual network (VNet). Once both VMs are created, we configure the IP address on the domain controller (DC-1), changing it from dynamic to static. This configuration will ensure that the client VM (Client-1) can use the domain controller (DC-1) as its DNS server and join its domain. 
</p>
<br />

<p>
<img src="https://i.imgur.com/SaM7cLB.jpg" height="80%" width="80%"/>
</p>
<p>
Next, we enter the domain controller (DC-1) through a remote desktop connection (RDP). Once inside, we disable the firewall for the domain, plus the private and public profiles. We do this by right-clicking the Windows symbol, selecting “run”, and typing wf.msc. In the Windows Defender Firewall window, we ensure the firewall is turned off for all profiles. 
</p>
<br />

<p>
<img src="https://i.imgur.com/NrDMeTl.png" height="80%" width="80%"/>
</p>
<p>
We’ll then change the DNS server on Client-1 to the static private IP address of DC-1 within the network settings within the Azure portal. In order to fully apply the new DNS configuration, we restart both virtual machines.
</p>
<br />

<p>
<img src="https://i.imgur.com/TgeIFuO.jpg" height="80%" width="80%"/>
</p>

<p>
<img src="https://i.imgur.com/OFxul1U.jpg" height="80%" width="80%"/>
</p>
<p>
Once both VMs have successfully restarted, we RDP into the Client-1 VM. Using PowerShell, we ping DC-1’s IP address. Since we disabled DC-1’s firewall, this should be successful, allowing DC-1 to respond to the ping. We then use the ipconfig /all command on Client-1 to confirm that DC-1 is configured as the DNS server for the virtual machine. 
</p>
<br />

<p>
<img src="https://i.imgur.com/q9rGJYV.jpg" height="80%" width="80%"/>
</p>
<p>
Next, on the domain controller (DC-1) we install Active Directory Domain Services using Server Manager.
</p>
<br />

<p>
<img src="https://i.imgur.com/3sSEBVO.jpg" height="80%" width="80%"/>
</p>
<p>
We then promote DC-1 to a Domain Controller and set up a new forest with the domain name called “mydomain.com”.
</p>
<br />

<p>
<img src="https://i.imgur.com/VDo9Pfv.jpg" height="80%" width="80%"/>
</p>
<p>
Next, we open Active Directory Users and Computers. We create two organizational units called _EMPLOYEES and _ADMINS. We do this by right-clicking mydomain.com, selecting “New” and choosing “Organizational Unit”.
</p>
<br />

<p>
<img src="https://i.imgur.com/z0DZHQe.jpg" height="80%" width="80%"/>
</p>
<p>
In the _ADMINS organizational unit (OU), we create a new user named Jane Doe with the username “jane_admin”. 
</p>
<br />

<p>
<img src="https://i.imgur.com/i7n4ktI.jpg" height="80%" width="80%"/>
</p>
<p>
Next, we add Jane Doe as a domain admin. We right-click the user, select Properties, navigate to the “Members Of” tab and click “Add”. In the “Enter object names” field, we type “domain admins” and press Enter to complete the process.
</p>
<br />

<p>
<img src="https://i.imgur.com/XfZmInx.png" height="80%" width="80%"/>
</p>
<p>
Then we log out of DC-1 and reconnect to it using RDP with the credentials “mydomain.com\jane_admin and the assigned password. This account will be used for all future logins into DC-1. 
</p>
<br />

<p>
<img src="https://i.imgur.com/KfDATZt.jpg" height="80%" width="80%"/>
</p>
<p>
Next, we join the domain from the Client-1 VM. We RDP into the system, right-click the Windows logo, select “System”, then “Rename this PC (Advanced)”. After that, we click “Change”, select “Domain” and enter the domain name “mydomain.com”. To apply the changes, the VM will then restart.
</p>
<br />

<p>
<img src="https://i.imgur.com/QgnHK2V.jpg" height="80%" width="80%"/>
</p>
<p>
We then go back to DC-1 and open Active Directory Users and Computers. We create another organizational unit named _CLIENTS under mydomain.com. 
</p>
<br />

<p>
<img src="https://i.imgur.com/3LKZ8Rf.jpg" height="80%" width="80%"/>
</p>
<p>
Next, we log into the Client-1 VM as jane_admin. We right-click the Windows logo, select “System”, then choose “Remote Desktop”. We then click “Select users that can remotely access this PC” and add domain users to allow them remote desktop (RDP) access to the VM.
</p>
<br />

<p>
<img src="https://i.imgur.com/igTqfA9.jpg" height="80%" width="80%"/>
</p>
<p>

We then use the script found here: https://github.com/joshmadakor1/AD_PS/blob/master/Generate-Names-Create-Users.ps1. We log into DC-1 as jane_admin and open PowerShell ISE as an administrator. We create a new file, paste the script into it, and execute. We observe as thousands of accounts are created automatically. 

</p>
<br />

<p>
<img src="https://i.imgur.com/4FSngfK.jpg" height="80%" width="80%"/>
</p>
<p>
After running the script, we open Active Directory Users and Computers to verify that the accounts have been created in the appropriate _EMPLOYEES organizational unit. 
</p>
<br />

<p>
<img src="https://i.imgur.com/4WGq6d0.jpg" height="80%" width="80%"/>
</p>
<p>
Lastly, we log into the Client-1 VM using one of the many accounts created by the script we ran earlier. We do this with the username and the default password of “Password1”. After logging in, we open PowerShell to verify that we are logged in as one of the script-created users. 
</p>
<br />
