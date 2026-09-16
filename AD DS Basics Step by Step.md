**Creating a Windows Server VM using Azure**
1. On the Azure portal, select "Create a resource" and then click "Create a virtual machine" from the list of services
2. Keep subscription option as default(Azure subscription 1) and name the resource group
3. Name the VM, select appropriate region and availability zone
4. Select image as Windows Server 2022 Datacenter: Azure Edition Hotpatch - x64 Gen 2 and select a size
5. Create admin account (remember this username and password)
6. Change Inbound ports to none since we will be connecting via Bastion
7. Read over the "Review + create" tab and create the VM

**Deploying Bastion**
1. On the top toolbar menu of the VM dashboard, select "Connect" and then "Connect via Bastion"
2. Using automatic config, deploy Bastion (this will take approx. 10 min)

**Setting Static IP for Server**
1. Set a static IP address since this server will be used as a domain controller
2. Select Networking>Network Settings from the VM dashboard's side bar
3. Click on the Network interface name to open the NIC settings
4. Open the default ipconfig1
5. Change Private IP address settings allocation from Dynamic to Static 
6. Keep the default IP address and record it (you'll need this information later)

**Connecting to VM**
1. Click "Connect via Bastion" from the VM dashboard
2. Select connection settings; Protocol: RDP, Port 3389
3. Authenticate using admin credentials
4. Click connect and spin up the VM for the first time

**Adding Roles and Features with Server Manager**
1. Select "Manage" from the top right corner of Server Manager and click "Add Roles and Features"
2. Select "Role based or feature-based Installation" as Installation Type
3. Select the VM from the server pool for Server Selection
4. Install the following Server Roles
	- Active Directory Domain Services
	- DHCP Server
	- DNS Server
	- Print and Document Services
	- Web Server (IIS)
5. Ensure "Group Policy Management" is selected on the next tab(Features)
6. Keep installation configs as default for each service and click Install
7. After installation, verify all services are visible from Server Manager and reboot server

**Promoting Server to Domain Controller**
1. Select AD DS in the Server Manager
2. Click the "More. ." button on the Configuration required notification
3. Click "Promote this server to a domain controller"
4. Select "Add a new forest" and name the root domain name "lab.local"
5. Create Directory Services Restore Mode password
6. Continue through configuration and click Install

**Creating an Organizational Unit and a User**
1. Open Active Directory Users and Computers (ADUC)
2. Right-click DC name and select New>Organizational Unit
3. Name and create OU (Branch 1)
4. Right-click Branch 1 and create another OU within called Users
5. Create two more OUs: Computers, Groups
6. Right-click "Users" and click New>User 
7. Create username and temporary password. Check "User must change password at next logon" then click "Finish" to create User

**Creating a Group**
1. Navigate to the "Groups" OU 
2. This OU should be empty. Right-click the empty space and select New>Group
3. Name the Group
4. Select "Global" for Group scope and "Security" for Group type
5. Double-click the group and navigate to the "Members" tab
6. Click "Add. ."
7. Enter the name of the previously created User and press the Enter key
8. Click "Apply"

**Copying a User**
1. Navigate to the Users OU
2. Right-click the User that was previously created and select "Copy. ."
3. From the Copy Object window, enter a new name for another User as well as new user logon name and password
4. Click finish
5. Double-click the new user and navigate to "Member of"
6. Here we will see our new user is already a member of our created Group OU because copying the user gave this new user the same set of permissions

**Resetting a password and Unlocking accounts**
1. Right-click a user and select "Reset Password. ."
2. The reset password window allows you to create a new password
3. Check "User must change password at next logon" so they can set their own password once they logon
4. You can also Unlock a user's account by checking that option from this window. Notice the "Account Lockout Status on this Domain Controller:" which tells you whether the account is locked or unlocked
5. Alternatively, double-click a User and navigate to the "Account" tab. Under User logon names, there is a checkbox to Unlock account. This tab will also tell you if the User is locked out when the account is locked

**Disabling and Enabling accounts**
1. Navigate to the "Account" tab on any User
2. Look for the list titled "Account options"
3. Scroll down until you see the option "Account is disabled" and check the box
4. Click Apply
5. Notice the icon for the listed User has changed to represent that the account is Disabled
6. Repeat steps 1-4 but uncheck the box "Account is disabled" to reenable the account

**Moving Users**
1. Build a new OU under the DC (separate from Branch 1) and name it Branch 2
2. Create the same OUs within Branch 2 as Branch 1 (Users, Computers, Groups)
3. Navigate to Branch 1>Users
4. Drag and drop a User from here to Branch 2>Users
5. AD DS will stop and ask if you're sure you want to do this because it can cause changes to the way the system currently works. For this example everything will be fine but be careful practicing this in real world scenarios
6. Click Yes
7. Navigate to Branch 2>Users and confirm the User was moved

**How to search for Objects**
1. Search for the Icon on the AD toolbar that has a page and a magnifying glass called "Find objects in AD DS" and click the icon
2. Notice the "Find:" options that allow you to filter what kind of object you're looking for as well as the "in:" option that allows you to select which OU you would like to search.
3. In order to find any User in the forest, select "Find: Users, Contacts, Groups in: Entire Directory"
4. Practice by searching one of the created Users
5. Navigate to the "Object" tab on the user you found to view the "Canonical name of object" which tells us the location of the User

**Creating a Client VM**
1. In order to join a client to the domain we will have to create another VM on Azure to use as the client
2. Follow the steps for "Creating a Windows Server VM using Azure" and be sure to select the same resource group the server VM is located in
3. Name this VM something like Client-VM or LAB-PC
4. We will be using the same image (Windows Server 2022 Datacenter: Azure Edition) since the Windows 11 Pro version requires a paid license to use but we will be treating this VM as a endpoint/User PC and NOT as a server
5. Navigate to the Networking Tab before continuing to "Review + create"
6. Make sure the selected Virtual network and Subnet are the same as the DC. (refer to the IP address of the DC and check its config from the Azure dashboard) 
7. Click "Review + create"
8. After successful deployment, navigate to the Overview of the new VM
9. Navigate to network settings and open the Network interface
10. Navigate to "DNS servers" and click Custom
11. Enter the IP address of the DC and click Apply
12. Connect via Bastion and spin up the VM

**Joining the client to the domain**
1. Make sure both VMs(PC and Server) are up and running
2. Navigate to Windows Settings
3. Click "System"
4. Click the "About" tab on the sidebar and then click "Advanced system settings" from "Related settings" 
5. This will open system properties, click the "Computer Name" tab
6. Notice the "Full computer name" and "Workgroup" 
7. Click "Change. ."
8. Under "Member of" select "Domain" and type in the name of your DC (we named the DC in Step 4 of "Promoting Server to Domain Controller)
9. If an error message appears stating an AC DC for the domain name could not be contacted, this points to a DNS error and we must troubleshoot(see below)
10. If connection is successful, you will be prompted to enter the username and password of the admin account for the DC
11. Enter credentials and you should receive a message "Welcome to the domain"

**DNS Troubleshooting**
1. Open command prompt and run `ipconfig /all`
2. Look for the DNS Servers IPV4 address. Does this address match that of our DC? If the answer is no then we must reconfigure the DNS Servers address
3. In the Windows start search bar type ncpa.cpl and select it from the start menu. This opens Network Connections in the control panel
4. Right click "Ethernet" and select Properties
5. Within the connections checklist, uncheck the IPV6 box
6. Click on IPV4 and click Properties
7. Under DNS, select Use the following DNS server addresses
8. Enter the IP address of your DC
9. Refer back to step 8 of "Joining the client to the domain"

**Verifying Domain connectivity**
1. Open command prompt on the PC VM
2. Enter command `ping (name of sever VM)`
3. You should receive a successful ping for all 4 packets
4. Go to the Server VM
5. Open ADUC
6. Navigate to the Computers OU that was automatically created when we created the AD DC(not the computers OU located in Branch 1 or 2)
7. The PC may not appear at first, but click the Refresh icon on the top toolbar and it should be visible
8. You can drag and drop the PC to the Computers OU of Branch 1 or Branch 2 for better organization

**Creating a GPO**
1. On the Windows Server VM, search "Group Policy Management" from the start menu and open it
2. Follow the side bar down to our domain name and under the domain right-click "Group Policy Objects" then click "New"
3. For lab purposes we will be disabling the domain firewall (this would be an unsafe practice in real-world production) so name this GPO "Disable Domain Firewall"
4. The GPO is then created and appears in the side bar. You can click the the GPO to view the default config (observe the different tabs like Scope and Settings)
5. Right-click this GPO on the side bar and click "Edit. ."
6. This opens the Group Policy Management Editor. Notice the two types of configurations: Computer Configuration and User Configuration. Computer configs apply settings to a machine while User configs apply settings to a specific user or group of users. For this GPO we will be applying it as a Computer Configuration
7. Notice under Computer Configuration on the side bar there are two objects: Policies and Preferences. Policies are settings that the user cannot change while Preferences are settings that are optional for the user. This GPO will be applied as a Policy
8. Navigate to Policies>Windows Settings>Security Settings>Windows Defender Firewall
9. Double Click the Windows Defender Firewall with Advanced Security object to open the firewall config
10. Click on Windows Defender Firewall Properties (You will notice the firewall is not configured at the domain level, but Windows will have this firewall turned on by default if you went and checked on our PC VM)
11. Click the drop down menu for "Firewall state" and select "Off"
12. Click OK and notice in the config: "Domain Profile: Windows Defender Firewall is turned off"
13. Close the Editor and click the "Refresh" icon on the top tool bar of the GPM
14. You can now see the configuration has been added under the Settings tab of this GPO, under Computer Configurations

**Testing and Verifying the GPO**
1. Click the "Scope" tab on the Disable Domain Firewall config
2. Under "Security Filtering" we want to remove the default "Authenticated Users" and then add our client PC by clicking "Add. ." 
3. Click the "Object Types. ." button since Computers are not selected by default and check the "Computers" box. Under "Enter the object name to select" type the exact name of our client PC. (If you can't remember the name, refer to ADUC by looking under our domain name on the left hand side. If you've followed the above steps it should be in either Branch 1>Computers or Branch 2>Computers. If you never moved it, then it will be under "Domain Name">Computers by default)
4. The client PC should now be added to the Security Filtering list
5. Now we will link the GPO by right clicking the Branch that the PC is located in and selecting "Link an existing GPO. ." and click "Disable Domain Firewall" then "OK"
6. You will notice under Disable Domain Firewall Scope that the Branch is now linked and the GPO will also show up under the Branch 1 object on the left had side bar
7. Switch to the PC VM and open command prompt
8. Run `gpupdate /force` (expected result is a successfully completed update message)
9. Then run `gpresult /r /scope: computer` to view a report of computer settings, applied GPOs, etc. Notice the GPO "Disable Domain Firewall" has been added
10. To verify the GPO actually works, open the start menu and search firewall and then open "Windows Defender" and notice "Domain network (active): Firewall is off."