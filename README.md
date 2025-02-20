# Active Directory Applications Overview

In this walkthrough, we will turn a virtual machine (running Windows Server) into a domain controller by activating Active Directory, to demonstrate how an IT worker can manage users and computers on a company network.

We will reuse the Windows 10 client workstation originally made in [the networking project](https://github.com/lorenzos-IT/networking-project).

## Technology used

* Windows 10 Pro
* Windows Server 2022
* Server Manager
* Active Directory Users and Computers (ADUC)
* Group Policy Management Console
* Event Viewer

## Walkthrough
### Setup - Windows Server (domain controller)
**Video version:** <https://youtu.be/tFLZmQEUVdU>  
![Screenshot](https://i.imgur.com/PqeOi0Y.png)  

1. Go to [the Microsoft Azure portal](https://portal.azure.com/) and log in with your Microsoft account.
2. Search for and open the Virtual Machines section, click _Create_ to make the virtual machine that will serve as the domain controller, with the following settings:
	* Select the Resource Group of the client workstation running Windows 10.
	* Change the zone if needed to match the region of the Resource Group of the Windows 10 client workstation.
	* Windows Server 2022, 64-bit, 21H2
	* 2 vCPU's and 8 GB RAM
	* Set a username and a password to log in.
	* Go to the Networking tab, and select the same Virtual Network as the Windows 10 client workstation.
3. Click _Review + Create_ and wait for this virtual machine to finish deploying.
4. Return to Virtual Machines, and click on the name of the virtual machine running Windows Server to open its cloud-based settings.
5. Click _Network Settings_, then the name of the _Network interface/IP configuration_. (It will look something like `dc-1833_z3 / ipconfig1`.)
6. Click the name of the _IP Settings_. (It will look something like `ipconfig1`.)
7. Under _Edit IP Configuration_ > _Private IP address settings_, set the _Allocation_ to Static, and click _Save_.
8. Return to Virtual Machines, and copy the domain controller's public IP address:
	1. Windows: Return to Microsoft Remote Desktop, paste the same IP address, and launch the virtual machine.
	2. Mac OS X: Click the Add (+) button in the top-right corner of the Windows App, paste the same IP address, and launch the virtual machine.
9. Log in with the credentials you set back in step 2.
10. Open Windows Defender Firewall, click _Advanced Settings_ in the left-hand column , then click _Windows Defender Firewall Properties_ in the middle column.
11. Set _Firewall state_ to Off for _Domain Profile_, _Private Profile_, and _Public Profile_, to be able to test connectivity using `ping`.*  

**I believe it would be safer to add or edit an inbound security rule to explicitly allow ICMP for step 11.*

### Setup - Windows 10 (client workstation)
**Video version:** <https://youtu.be/2K9TVFaAdCw>  
![Screenshot](https://i.imgur.com/u82neKp.png)  

1. Return to Virtual Machines, click on the name of the Windows Server virtual machine to open its cloud-based settings, then copy its private IP address.
2. Go back to Virtual Machines again, then click on the name of the virtual machine running Windows 10 to open its cloud-based settings.
3. Click _Network Settings_, then the name of the _Network interface/IP configuration_. (It will look something like `dc-1833_z3 / ipconfig1`.)
4. Click _DNS servers_ in the left column, then click _Custom_ under _DNS servers_ to the right.
5. Paste the private IP address under _DNS server_, then click _Save_.
6. Return to Virtual Machines, tick the box for the client workstation, and click _Restart_.
7. Click _Refresh_ until the status of the Windows 10 machine changes from _Restarting_ to _Running_.
8. Log in with the credentials set for the virtual machine running Windows 10.
9. In PowerShell, run `ping` with the private IP address. (It will look something like `ping 10.0.0.5`.)
10. If the ping is successful, run `ipconfig -all`, then review the output. The DNS settings should reflect the domain controller's IP address.

### Installing Active Directory Components
**Video version:** <https://youtu.be/PI-NHKitNgA>  
![Screenshot](https://i.imgur.com/BPcVn5T.png)  

1. Turn the domain controller back on:
	1. Windows: Open Microsoft Remote Desktop, select the appropriate IP address, and log in.
	2. Mac OS X: Open the Windows app, double-click the appropriate tile, and log in.
2. In Server Manager, click _2. Add roles and features_, then click _Next_ for the default selections until you reach the _Server Roles_ panel.
3. Click on _Active Directory Domain Servers_, then _Add Features_.
4. Keep clicking _Next_ under you reach the _Confirmation_ panel, tick the option to restart the server automatically if required, then click _Install_.
5. When the feature installation finishes successfully, click _Close_.
6. Open the post-deployment configuration panel by clicking the flag icon with an exclamation point (`!`) in the top-right corner.
7. Click _Promote this server to a domain controller_.
8. Under _Active Directory Domain Services Configuration Wizard_ > _Deployment Configuration_, select _Add a new forest_ and decide on the _root domain name_. (The latter will look something like `mnl-it.com`.)
9. In the _Domain Controller Options_ panel, decide on the DSRM (Directory Services Restore Mode) password.
10. Keep clicking _Next_ under you reach the _Prerequisites Check_ panel, wait for the verification to finish, then click _Install_.
11. Click _Close_ when you see the successful server configuration alert, then the machine will restart shortly.
12. Log back into the domain controller virtual machine this time using the `username@domain` syntax. (This will look something like `jcruz@mnl-it.com`.)

### Creating an Admin in the Domain
**Video version:** <https://youtu.be/BDGOiHCqiIs>  
![Screenshot](https://i.imgur.com/akPDLtp.png)  

1. Open Active Directory Users and Computers by looking it up from the search bar or by running `dsa.msc`.
2. From the left sidebar, double-click the domain name of the forest you made earlier.
3. Right-click on an empty space in the right panel and create two Organizational Units (OUs) named `_EMPLOYEES` and `_ADMINS`.
4. Right-click on the domain name again, then click _Refresh_.
5. From the left sidebar, double-click `_ADMINS`, then right-click on an empty space in the right panel and create a user named `Jane Cruz`.
6. Right-click on `Jane Cruz`, click _Properties_, click _Member Of_, and click _Add_.
7. Type `Domain Admins` as the object name to select, click _Check Names_, click _OK_, click _Apply_, then click _OK_ again to make this user an admin.
8. Log out of the domain controller, then log back in as the admin account you just made.

### Adding a Workstation to the Domain
**Video version:** <https://youtu.be/UeObB57A4PQ>  
![Screenshot](https://i.imgur.com/aek7Gzh.png)  

1. Log in to the client workstation with your original admin account.
2. Type _About_ in the taskbar's search and launch _About your PC_.
3. In the right sidebar under _Related settings_, click _Rename this PC (advanced)_.
4. A _System Properties_ dialog box will open. Under the _Computer Name_ tab, click _Change_.
5. _Computer Name/Domain Changes_ will then open. Under _Member of_, type your chosen domain name under _Domain_, click _OK_, click _Apply_, then click _OK_ again to make this machine join the domain and restart.
6. Log back in to the domain controller, open Active Directory Users and Computers, and look for the name of the client workstation.
7. Make a new OU named `_CLIENTS` and drag the client workstation's entry in ADUC into this new OU.

### Granting Remote Desktop Access to Users
**Video version:** <https://youtu.be/1_TDFAwvg64>  
![Screenshot](https://i.imgur.com/1GbMPes.png)  

1. Log back in to the client workstation.
2. Search for _System Properties_ in the taskbar's search and launch _About your PC_.
3. From the left sidebar, click Remote Desktop, then under _User accounts_, click _Select users that can remotely access this PC_.
4. The _Remote Desktop Users_ dialog box will appear. Click _Add_.
5. Type `Domain Users` as the object name to select, click _Check Names_, click _OK_, then click _OK_ again to also allow users access to Remote Desktop for the client workstation.

### Creating Multiple Users and Logging in as One of Them
**Video version:** <https://youtu.be/CdL8RZgyl6k>  
![Screenshot](https://i.imgur.com/GCZ2ZOo.png)  

1. Log back into the domain controller.
2. Run Windows PowerShell ISE as an administrator.
3. Press `Control-N` to create a new file, then copy the contents of [this Powershell script from GitHub](https://github.com/joshmadakor1/AD_PS/blob/master/Generate-Names-Create-Users.ps1) and paste it inside the file you just made.
4. In the script, edit the variable `NUMBER_OF_ACCOUNTS_TO_CREATE` in line 3 to a more reasonable number of users. (Maybe `PASSWORD_FOR_USERS` in line 2 to a more secure password as well.)
5. Click the Run button (looks like the green play button) and observe the users being created in ADUC under the `_EMPLOYEES` OU.
6. When the script is finished, log into the client workstation as one of the newly-created user accounts, then log out.

### Using Group Policy Management Editor to Decide How Many Failed Password Attempts Allowed Before Account is Locked
**Video version:** <https://youtu.be/piKyj66Ig1g>  
![Screenshot](https://i.imgur.com/DmD9K5L.png)  

1. On the client workstation, log back in as one of the new users, but this time with the wrong password deliberately thrice in a row.
2. In the domain controller, open the Group Policy Management Console by looking it up from the search bar or by running `gpmc.msc`.
3. Right-click the existing group policy object (GPO) named `Default Domain Policy` that's listed directly under the forest (in this example, `mnl-it.com`), then click _Edit_.
4. The Group Policy Management Editor will then open. Go to _Computer Configuration_ > _Policies_ > _Windows Settings_ > _Security Settings_ > _Account Policies_ > _Account Lockout Policy_.
5. Right-click on _Account Lockout Threshold_ and set it to 5 invalid login attempts. Suggested value changes for the other policies, _Account Lockout Duration_, _Allow_Administrator Account Lockout_, and _Reset Account Lockout Counter After_, will then appear. Click _OK_, _Apply_, and _OK_.
6. Return to the Group Policy Management Console, right-click the forest's name, then click _Link an Existing GPO_. Notice that the existing GPO `Default Domain Policy` is already linked.
7. Return to the client workstation and log in as an admin.
8. The policy updated on the server will automatically propagate out to clients, but you can force this to happen sooner by opening PowerShell and entering the command `gpupdate /force`.
9. Open the Resultant Set of Policy (`rsop.msc`) tool on the client workstation and wait for it to complete processing.
10. Go to _Computer Configuration_ > _Policies_ > _Windows Settings_ > _Security Settings_ > _Account Policies_ > _Account Lockout Policy_, and notice that the _Account Lockout Policy_ now matches what you updated it to on the domain controller.
11. Log out of the admin account on the client workstation.
12. Log back in to the client workstation as the same new user with a wrong password another three times. Notice that on the fifth attempt overall, an error message will display saying that your account is locked.
13. Return to the domain controller, open ADUC, then refresh and open the `_EMPLOYEES` OU.
14. Right-click the name of the new user you logged in as, then click _Reset Password_.
15. The account's lockout status will appear in the resulting dialog box as `Locked out`. With _Unlock the user's account_ ticked, set a new password for the account, and click _OK_ twice.
16. Return to the client workstation, and you will then be able to log in with your newly-reset password.

### Disabling and Re-Enabling of an Account
**Video version:** <https://youtu.be/1_xjrEg4SvM>  
![Screenshot](https://i.imgur.com/rvUT67h.png)  

1. Open ADUC in the domain controller, then refresh and open the `_EMPLOYEES` OU.
2. Right-click the name of the new user you logged in as, then click _Disable Account_.
3. Log back in to the client workstation as the same new user. Notice that an error message will display saying that your account is disabled.
4. Back in ADUC, right-click the name of the new user again, then click _Enable Account_.
5. You will then be able to log in to the client workstation as the same user.*

**In the real world, you should verify the identity of a person before unlocking an account, according to your company's process and to prevent social engineering attacks.*

### Observing Logs in Event Viewer
**Video version:** <https://youtu.be/n1-U1v28KnM>  
![Screenshot](https://i.imgur.com/Gf7BTCt.png)  

1. Open Event Viewer in the domain controller by looking it up from the search bar or by running `eventvwr.msc`.
2. In the left sidebar, open _Windows Logs_, double-click _Security_, and wait for its logs to load.
3. Review _Audit Success_ and _Audit Failure_ logs for _Logon_ tasks. Notice that for _Audit Failure_ logs, the _General_ panel shows your user account's name and the _Error Code_ that prevented logging in.
4. Return to the client workstation and log in as an admin, then go to Event Viewer > _Windows Logs_ > _Security_.
5. Go over the _Audit Success_ and _Audit Failure_ logs for _Logon_ tasks. Notice that the same logs on the client workstation will show the _Failure Reason_ in addition to the error code that was shown on the domain controller.

