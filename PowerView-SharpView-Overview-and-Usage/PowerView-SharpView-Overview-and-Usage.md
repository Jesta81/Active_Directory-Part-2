## PowerView/SharpView Overview & Usage


- [SharpView](https://github.com/dmchell/SharpView) is a .NET port of [PowerView](https://github.com/PowerShellMafia/PowerSploit/blob/dev/Recon/PowerView.ps1), one of many tools contained within the now deprecated [PowerSploit](https://github.com/PowerShellMafia/PowerSploit/tree/master/Recon) offensive PowerShell toolkit. This [Read the Docs page](https://powersploit.readthedocs.io/en/latest/Recon/) explains the function naming schema and provides information about the various parameters that can be passed to each function. 


- Note: Since writing this module, we noticed that BC-Security has started pushing updates to PowerView as part of their [Empire](https://github.com/BC-SECURITY/Empire) project. This course still uses the Development PowerView module out of PowerSploit's GitHub, but by the end of the year, we plan to migrate this to the version that Empire uses. 


- In the past, PowerShell was the scripting language of choice for offensive tools, but it has become more security transparent, with better detection optics available for both consumer and enterprise-level endpoint protection products. For this reason, offensive security practitioners have evolved their tradecraft to mitigate improved security monitoring capabilities and have ported their tooling to C# inline, which is less security transparent. While **PowerView** is no longer officially maintained, it is still an extremely powerful AD enumeration tool and can be useful when performing engagements where stealth is not a requirement. It also remains useful for defenders who are looking to gain a better understanding of their AD environment. We will cover the history and general usage of **PowerView**, but this module (and related modules) will focus on **SharpView** in line with current .NET tradecraft to be more applicable to real-life, modern engagements. We will cover general **PowerView and SharpView** usage in this module because both still have their uses, depending on the situation. 


- Both tools can perform enumeration, gain situational awareness, and perform attacks within a Windows domain. **PowerView** utilizes PowerShell AD hooks and Win32 API functions, and, among other functions, replaces a variety of net commands called by the built-in Windows tools. **SharpView** is a .NET port that provides all of the **PowerView** functions and arguments in a .NET assembly. One major difference between **PowerView and SharpView** is the ability to pipe commands. **SharpView** uses strings instead of PowerShell objects. Therefore we cannot specify properties using **Select or Select-Object**, to parse the output or select specific AD objects as easily. 


![CMD](/PowerView-SharpView-Overview-and-Usage/images/account.png) 


- Here we can see that a command similar to **net accounts** can be performed with the **PowerView or SharpView command Get-DomainPolicy**. 


![CMD](/PowerView-SharpView-Overview-and-Usage/images/domain-policy.png) 


- The functionality of both tools can be grouped into different "buckets". While we will not cover every single function in this section, we will cover some of the most important ones under each. Both tools use the same functions and arguments, but the output can differ. This [Read the Docs documentation](https://powersploit.readthedocs.io/en/latest/Recon/) provides an in-depth description of each function and command syntax and various examples for how the functions can be used. 


### Misc Functions


- The misc functions offer various useful tools such as converting UAC values, SID conversion, user impersonation, Kerberoasting, and more. The entire list of functions with explanations from the tool documentation is as follows. 


	Export-PowerViewCSV             -   thread-safe CSV append
	Resolve-IPAddress               -   resolves a hostname to an IP
	ConvertTo-SID                   -   converts a given user/group name to a security identifier (SID)
	Convert-ADName                  -   converts object names between a variety of formats
	ConvertFrom-UACValue            -   converts a UAC int value to human readable form
	Add-RemoteConnection            -   pseudo "mounts" a connection to a remote path using the specified credential object
	Remove-RemoteConnection         -   destroys a connection created by New-RemoteConnection
	Invoke-UserImpersonation        -   creates a new "runas /netonly" type logon and impersonates the token
	Invoke-RevertToSelf             -   reverts any token impersonation
	Get-DomainSPNTicket             -   request the kerberos ticket for a specified service principal name (SPN)
	Invoke-Kerberoast               -   requests service tickets for kerberoast-able accounts and returns extracted ticket hashes
	Get-PathAcl                     -   get the ACLs for a local/remote file path with optional group recursion


- We can use **SharpView or PowerView** to convert a username to the corresponding [SID](https://en.wikipedia.org/wiki/Security_Identifier). And vise versa. 


![CMD](/PowerView-SharpView-Overview-and-Usage/images/sid.png) 


- When we enumerate UAC values using the useraccountcontrol value, the values are displayed back to us as numerical values, not in a human-readable format. We can use the ConvertFrom-UACValue function. If we add the -showall property, all common UAC values are shown, and the ones that are set for the user are marked with a +. This can be saved as a reference on a cheat sheet for future engagements. 


![CMD](/PowerView-SharpView-Overview-and-Usage/images/uac.png) 


### Domain / LDAP Functions


	Get-DomainDNSZone               -   enumerates the Active Directory DNS zones for a given domain
	Get-DomainDNSRecord             -   enumerates the Active Directory DNS records for a given zone
	Get-Domain                      -   returns the domain object for the current (or specified) domain
	Get-DomainController            -   return the domain controllers for the current (or specified) domain
	Get-Forest                      -   returns the forest object for the current (or specified) forest
	Get-ForestDomain                -   return all domains for the current (or specified) forest
	Get-ForestGlobalCatalog         -   return all global catalogs for the current (or specified) forest
	Find-DomainObjectPropertyOutlier-   inds user/group/computer objects in AD that have 'outlier' properties set
	Get-DomainUser                  -   return all users or specific user objects in AD
	New-DomainUser                  -   creates a new domain user (assuming appropriate permissions) and returns the user object
	Set-DomainUserPassword          -   sets the password for a given user identity and returns the user object
	Get-DomainUserEvent             -   enumerates account logon events (ID 4624) and Logon with explicit credential events
	Get-DomainComputer              -   returns all computers or specific computer objects in AD
	Get-DomainObject                -   returns all (or specified) domain objects in AD
	Set-DomainObject                -   modifies a given property for a specified active directory object
	Get-DomainObjectAcl             -   returns the ACLs associated with a specific active directory object
	Add-DomainObjectAcl             -   adds an ACL for a specific active directory object
	Find-InterestingDomainAcl       -   finds object ACLs in the current (or specified) domain with modification rights set to non-built in objects
	Get-DomainOU                    -   search for all organization units (OUs) or specific OU objects in AD
	Get-DomainSite                  -   search for all sites or specific site objects in AD
	Get-DomainSubnet                -   search for all subnets or specific subnets objects in AD
	Get-DomainSID                   -   returns the SID for the current domain or the specified domain
	Get-DomainGroup                 -   return all groups or specific group objects in AD
	New-DomainGroup                 -   creates a new domain group (assuming appropriate permissions) and returns the group object
	Get-DomainManagedSecurityGroup  -   returns all security groups in the current (or target) domain that have a manager set
	Get-DomainGroupMember           -   return the members of a specific domain group
	Add-DomainGroupMember           -   adds a domain user (or group) to an existing domain group, assuming appropriate permissions to do so
	Get-DomainFileServer            -   returns a list of servers likely functioning as file servers
	Get-DomainDFSShare              -   returns a list of all fault-tolerant distributed file systems for the current (or specified) domain


- The LDAP functions provide us with a wealth of useful commands. The **Get-Domain** function will provide us with information about the domain, such as the name, any child domains, a list of domain controllers, domain controller roles, and more.


![CMD](/PowerView-SharpView-Overview-and-Usage/images/child.png) 


- We can begin to get the lay of the land with the **Get-DomainOU** function and return the names of all Organizational Units (OUs), which can help us map out the domain structure. We can enumerate these names with **SharpView**. 


![CMD](/PowerView-SharpView-Overview-and-Usage/images/ou.png) 


![CMD](/PowerView-SharpView-Overview-and-Usage/images/ou-2.png) 


- We can gather information about domain users with the **Get-DomainUser** function and specify properties such as **PreauthNotRequired** to try planning out attacks. 


![CMD](/PowerView-SharpView-Overview-and-Usage/images/kerberos.png) 


- We can also begin gathering information about individual hosts using the **Get-DomainComputer function**. 


![CMD](/PowerView-SharpView-Overview-and-Usage/images/computer.png) 

![CMD](/PowerView-SharpView-Overview-and-Usage/images/computer-2.png) 


### GPO functions


	Get-DomainGPO                           -   returns all GPOs or specific GPO objects in AD
	Get-DomainGPOLocalGroup                 -   returns all GPOs in a domain that modify local group memberships through 'Restricted Groups' or Group Policy preferences
	Get-DomainGPOUserLocalGroupMapping      -   enumerates the machines where a specific domain user/group is a member of a specific local group, all through GPO correlation
	Get-DomainGPOComputerLocalGroupMapping  -   takes a computer (or GPO) object and determines what users/groups are in the specified local group for the machine through GPO correlation
	Get-DomainPolicy                        -   returns the default domain policy or the domain controller policy for the current domain or a specified domain/domain controller


![CMD](/PowerView-SharpView-Overview-and-Usage/images/gpo.png) 


- We can also determine which GPOs map back to which hosts. 


![CMD](/PowerView-SharpView-Overview-and-Usage/images/gpo-2.png) 


### Computer Enumeration Functions


	Get-NetLocalGroup                   -   enumerates the local groups on the local (or remote) machine
	Get-NetLocalGroupMember             -   enumerates members of a specific local group on the local (or remote) machine
	Get-NetShare                        -   returns open shares on the local (or a remote) machine
	Get-NetLoggedon                     -   returns users logged on the local (or a remote) machine
	Get-NetSession                      -   returns session information for the local (or a remote) machine
	Get-RegLoggedOn                     -   returns who is logged onto the local (or a remote) machine through enumeration of remote registry keys
	Get-NetRDPSession                   -   returns remote desktop/session information for the local (or a remote) machine
	Test-AdminAccess                    -   rests if the current user has administrative access to the local (or a remote) machine
	Get-NetComputerSiteName             -   returns the AD site where the local (or a remote) machine resides
	Get-WMIRegProxy                     -   enumerates the proxy server and WPAD contents for the current user
	Get-WMIRegLastLoggedOn              -   returns the last user who logged onto the local (or a remote) machine
	Get-WMIRegCachedRDPConnection       -   returns information about RDP connections outgoing from the local (or remote) machine
	Get-WMIRegMountedDrive              -   returns information about saved network mounted drives for the local (or remote) machine
	Get-WMIProcess                      -   returns a list of processes and their owners on the local or remote machine
	Find-InterestingFile                -   searches for files on the given path that match a series of specified criteria



- The computer enumeration functions can gather information about user sessions, test for local admin access, search for file shares and interesting files, and more. The **Test-AdminAccess function** can check if our current user has local admin rights on any remote hosts. 


![CMD](/PowerView-SharpView-Overview-and-Usage/images/admin.png) 


- We can use the **Net-Share function** to enumerate open shares on a remote computer. Shares can hold a wealth of information, and the importance of enumerating file shares should not be overlooked. 


![CMD](/PowerView-SharpView-Overview-and-Usage/images/netshare.png) 


### Threaded 'Meta'-Functions


	Find-DomainUserLocation             -   finds domain machines where specific users are logged into
	Find-DomainProcess                  -   finds domain machines where specific processes are currently running
	Find-DomainUserEvent                -   finds logon events on the current (or remote domain) for the specified users
	Find-DomainShare                    -   finds reachable shares on domain machines
	Find-InterestingDomainShareFile     -   searches for files matching specific criteria on readable shares in the domain
	Find-LocalAdminAccess               -   finds machines on the local domain where the current user has local administrator access
	Find-DomainLocalGroupMember         -   enumerates the members of specified local group on machines in the domain


- The 'meta' functions can be used to find where domain users are logged in, look for specific processes on remote hosts, find domain shares, find files on domain shares, and test where our current user has local admin rights. We can use the **Find-DomainUserLocation function** to find domain machines that users are logged into. 


![CMD](/PowerView-SharpView-Overview-and-Usage/images/location.png) 


### Domain Trust Functions


	Get-DomainTrust                     -   returns all domain trusts for the current domain or a specified domain
	Get-ForestTrust                     -   returns all forest trusts for the current forest or a specified forest
	Get-DomainForeignUser               -   enumerates users who are in groups outside of the user's domain
	Get-DomainForeignGroupMember        -   enumerates groups with users outside of the group's domain and returns each foreign member
	Get-DomainTrustMapping              -   this function enumerates all trusts for the current domain and then enumerates all trusts for each domain it finds


- The domain trust functions provide us with the tools we need to enumerate information that can be used to mount cross-trust attacks. The most basic of these commands, **Get-DomainTrust** will return all domain trusts for our current domain.


![CMD](/PowerView-SharpView-Overview-and-Usage/images/trust.png) 
