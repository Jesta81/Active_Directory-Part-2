## Enumerating AD Groups


- Armed with the domain user information, it is next important to gather AD group information to see what privileges members of a group may have and even find nested groups or issues with group membership that could lead to unintended rights. 


### Domain Groups

- A quick check shows that our target domain, **INLANEFREIGHT.LOCAL has 72 groups**. 


![AD Groups](/Enumerating-AD-Groups/images/group-count.png) 

	PS C:\htb> Get-DomainGroup -Properties Name


- Let's grab a full listing of the group names. Many of these are built-in, standard AD groups. The presence of some group shows us that Microsoft Exchange is present in the environment. An Exchange installation adds several groups to AD, some of which such as Exchange Trusted Subsystem and Exchange Windows Permissions are considered [high-value targets](https://github.com/gdedrouas/Exchange-AD-Privesc) due to the permissions that membership in these groups grants a user or computer. Other groups such as Protected Users, LAPS Admins, Help Desk, and Security Operations should be noted down for review.

We can use **Get-DomainGroupMember** to examine group membership in any given group. Again, when using the **SharpView** function for this, we can pass the -Help flag to see all of the parameters that this function accepts. 


### Group Membership


![SharpHound](/Enumerating-AD-Groups/images/sharphound-help.png) 


	PS C:\htb> .\SharpView.exe Get-DomainGroupMember -Help

	Get_DomainGroupMember -Identity <String[]> -DistinguishedName <String[]> -SamAccountName <String[]> -Name <String[]> -MemberDistinguishedName <String[]> -MemberName <String[]> -Domain <String> -Recurse <Boolean> -RecurseUsingMatchingRule <Boolean> -LDAPFilter <String> -Filter <String> -SearchBase <String> -ADSPath <String> -Server <String> -DomainController <String> -SearchScope <SearchScope> -ResultPageSize <Int32> -ServerTimeLimit <Nullable`1> -SecurityMasks <Nullable`1> -Tombstone <Boolean> -Credential <NetworkCredential>


- A quick examination of the Help Desk group shows us that there are two members. 


![SharpHound](/Enumerating-AD-Groups/images/helpdesk.png) 

	PS C:\htb> .\SharpView.exe Get-DomainGroupMember -Identity 'Help Desk'



### Protected Groups


- Next, we can look for all AD groups with the **AdminCount attribute set to 1**, signifying that this is a protected group. 


![SharpHound](/Enumerating-AD-Groups/images/protected.png) 


![SharpHound](/Enumerating-AD-Groups/images/protected-2.png) 


	PS C:\> .\SharpView.exe Get-DomainGroup -AdminCount
	PS C:\> .\SharpView.exe Get-DomainGroup -AdminCount | findstr /b "name"
	PS C:\> .\SharpView.exe Get-DomainGroup -AdminCount | findstr /b "memberof"



#### Managed Security Groups


- Another important check is to look for any managed security groups. These groups have delegated non-administrators the right to add members to AD security groups and [distribution groups](https://docs.microsoft.com/en-us/exchange/recipients-in-exchange-online/manage-distribution-groups/manage-distribution-groups) and is set by modifying the **managedBy attribute**. This check looks to see if a group has a manager set and if the user can add users to the group. This could be useful for lateral movement by gaining us access to additional resources. First, let's take a look at the list of managed security groups. 


![SharpHound](/Enumerating-AD-Groups/images/managed.png) 


	PS C:\> Find-ManagedSecurityGroups | select GroupName
	PS C:\> Find-ManagedSecurityGroups


- Next, let's look at the Security Operations group and see if the group has a manager set. We can see that the user joe.evans is set as the group manager.


![SharpHound](/Enumerating-AD-Groups/images/managed-2.png) 


	PS C:\> Get-DomainManagedSecurityGroup | select GroupName, ManagerName


### Generic Write Abuse / Exploit


- Enumerating the ACLs set on this group, we can see that this user has **GenericWrite privileges** meaning that this user can modify group membership (add or remove users). If we gain control of this user account, we can add this account or any other account that we control to the group and inherit any privileges that it has in the domain. 


![SharpHound](/Enumerating-AD-Groups/images/sid.png) 

	PS C:\Tools> ConvertTo-SID joe.evans
	S-1-5-21-2974783224-3764228556-2640795941-1238


![SharpHound](/Enumerating-AD-Groups/images/sid-2.png) 


	> $sid = ConvertTo-SID joe.evans
	> Get-DomainObjectAcl -Identity 'Security Operations' | ?{$_.SecurityIdentifier -eq $sid}



### Local Groups


- It is also important to check local group membership. Is our current user local admin or part of local groups on any hosts? We can get a list of the local groups on a host using **Get-NetLocalGroup**.


![SharpHound](/Enumerating-AD-Groups/images/local.png) 


	> Get-NetLocalGroup -Computer WS01 | select GroupName


- We can also enumerate the local group members on any given host using the **Get-NetLocalGroupMember** function.



![SharpHound](/Enumerating-AD-Groups/images/localgroup.png) 

	> SharpView.exe Get-NetLocalGroup -ComputerName WS01


![SharpHound](/Enumerating-AD-Groups/images/localmember.png) 

	> SharpView.exe Get-NetLocalGroupMember -ComputerName WS01


- We see one non-RID 500 user in the local administrators group and use the Convert-SidToName function to convert the SID and reveal the harry.jones user. 


![SharpHound](/Enumerating-AD-Groups/images/sid-3.png) 

- We use this same function to check all the hosts that a given user has local admin access, though this can be done much quicker with another PowerView/SharpView function that we will cover later in this module. 


![SharpHound](/Enumerating-AD-Groups/images/access.png) 


	PS C:\Tools> $sid = Convert-NameToSid harry.jones
	PS C:\Tools> $computers = Get-DomainComputer -Properties dnshostname | select -ExpandProperty dnshostname
	PS C:\Tools> foreach ($line in $computers) {Get-NetLocalGroupMember -ComputerName $line | ? {$_.SID -eq $sid}}


