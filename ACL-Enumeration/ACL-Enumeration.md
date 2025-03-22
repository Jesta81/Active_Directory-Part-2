## ACL Enumeration 

Let's jump into enumerating ACLs using PowerView and walking through some graphical representations using BloodHound. We will then cover a few scenarios/attacks where the ACEs we enumerate can be leveraged to gain us further access in the internal environment. 


### Enumerating ACLs with PowerView 

We can use PowerView to enumerate ACLs, but the task of digging through all of the results will be extremely time-consuming and likely inaccurate. For example, if we run the function **Find-InterestingDomainAcl** we will receive a massive amount of information back that we would need to dig through to make any sense of: 

#### Using Find-InterestingDomainAcl

![AD Enumeration](/ACL-Enumeration/images/find-acl.png) 

`PS C:\> Find-InterestingDomainAcl`

If we try to dig through all of this data during a time-boxed assessment, we will likely never get through it all or find anything interesting before the assessment is over. Now, there is a way to use a tool such as PowerView more effectively -- by performing targeted enumeration starting with a user that we have control over. Let's focus on the user **wley**, which we obtained after solving the last question in the **LLMNR/NBT-NS Poisoning - from Linux section**. Let's dig in and see if this user has any interesting ACL rights that we could take advantage of. We **first need to get the SID of our target** user to search effectively. 


```
PS C:\> Import-Module .\PowerView.ps1
PS C:\> $sid = Convert-NameToSid wley
```


![AD Enumeration](/ACL-Enumeration/images/sid.png) 


We can then use the **Get-DomainObjectACL** function to perform our targeted search. In the below example, we are using this function to find all domain objects that our user has rights over by mapping the user's SID using the **$sid** variable to the **SecurityIdentifier** property which is what tells us who has the given right over an object. One important thing to note is that if we search without the flag **ResolveGUIDs**, we will see results like the below, where the right **ExtendedRight** does not give us a clear picture of what ACE entry the user **wley has over damundsen**. This is because the **ObjectAceType property is returning a GUID value that is not human readable.** 


### Using Get-DomainObjectACL 

`C:\> Get-DomainObjectACL -Identity * | ? {$_.SecurityIdentifier -eq $sid}`


We could Google for the GUID value **00299570-246d-11d0-a768-00aa006e0529** and uncover [this page](https://docs.microsoft.com/en-us/windows/win32/adschema/r-user-force-change-password) showing that the user has the right to force change the other user's password. Alternatively, we could do a reverse search using PowerShell to map the right name back to the GUID value. 

**Note that if PowerView has already been imported, the cmdlet shown below will result in an error. Therefore, we may need to run it from a new PowerShell session.** 

![AD Enumeration](/ACL-Enumeration/images/domain-object-acl.png) 

#### Performing a Reverse Search & Mapping to a GUID Value 

```

PS C:\> $guid = "00299570-246d-11d0-a768-00aa006e0529" 
PS C:\> Get-ADObject -SearchBase "CN=Extended-Rights,$((Get-ADRootDSE).ConfigurationNamingContext)" -Filter {ObjectClass -like 'ControlAccessRight'} -Properties * |Select Name,DisplayName,DistinguishedName,rightsGuid| ?{$_.rightsGuid -eq $guid} | fl 

```

![AD Enumeration](/ACL-Enumeration/images/guid.png) 


#### Using the Resolve-GUIDs Flag 

`PS C:\> Get-DomainObjectACL -ResolveGUIDs -Identity * | ? {$_.SecurityIdentifier -eq $sid}`


![AD Enumeration](/ACL-Enumeration/images/resolve-guids.png) 


**Why did we walk through this example when we could have just searched using ResolveGUIDs first?**

It is essential that we understand what our tools are doing and have alternative methods in our toolkit in case a tool fails or is blocked. Before moving on, let's take a quick look at how we could do this using the [Get-Acl](https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.security/get-acl?view=powershell-7.2) and [Get-ADUser](https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.security/get-acl?view=powershell-7.2) cmdlets which we may find available to us on a client system. Knowing how to perform this type of search without using a tool such as PowerView is greatly beneficial and could set us apart from our peers. We may be able to use this knowledge to achieve results when a client has us work from one of their systems, and we are restricted down to what tools are readily available on the system without the ability to pull in any of our own.

This example is not very efficient, and the command can take a long time to run, especially in a large environment. It will take much longer than the equivalent command using PowerView. In this command, we've first made a list of all domain users with the following command: 


### Creating a List of Domain Users 

`PS C:\> Get-ADUser -Filter * | Select-Object -ExpandProperty SamAccountName > ad_users.txt` 

![AD Enumeration](/ACL-Enumeration/images/users.png) 

We then read each line of the file using a **foreach loop**, and use the **Get-Acl** cmdlet to retrieve ACL information for each domain user by feeding each line of the **ad_users.txt file to the Get-ADUser** cmdlet. We then select just the **Access property**, which will give us information about access rights. Finally, we set the **IdentityReference property to the user we are in control of (or looking to see what rights they have), in our case, wley.** 

```
PS C:\htb> foreach($line in [System.IO.File]::ReadLines("C:\Users\htb-student\Desktop\ad_users.txt")) {get-acl  "AD:\$(Get-ADUser $line)" | Select-Object Path -ExpandProperty Access | Where-Object {$_.IdentityReference -match 'INLANEFREIGHT\\wley'}}
```

![AD Enumeration](/ACL-Enumeration/images/foreach-loop-users.png) 


Once we have this data, we could follow the same methods shown above to convert the GUID to a human-readable format to understand what rights we have over the target user.

So, to recap, we started with the user **wley and now have control over the user damundsen via the User-Force-Change-Password** extended right. Let's use Powerview to hunt for where, if anywhere, control over the **damundsen** account could take us. 


#### Further Enumeration of Rights Using damundsen 


![AD Enumeration](/ACL-Enumeration/images/sid2.png) 


Now we can see that our user **damundsen has GenericWrite privileges over the Help Desk Level 1 group**. This means, among other things, that we can add any user (or ourselves) to this group and inherit any rights that this group has applied to it. A search for rights conferred upon this group does not return anything interesting.

Let's look and see if this group is nested into any other groups, remembering that nested group membership will mean that any users in group A will inherit all rights of any group that group A is nested into (a member of). A quick search shows us that the **Help Desk Level 1 group is nested into the Information Technology group**, meaning that we can **obtain any rights that the Information Technology group grants to its members if we just add ourselves to the Help Desk Level 1 group where our user damundsen has GenericWrite privileges.** 

`PS C:\> Get-DomainGroup -Identity "Help Desk Level 1" | select memberof`


![AD Enumeration](/ACL-Enumeration/images/nested.png) 


This is a lot to digest! Let's recap where we're at: 

- We have control over the user **wley** whose hash we retrieved earlier in the module (assessment) using Responder and cracked offline using Hashcat to reveal the cleartext password value 

- We enumerated objects that the user **wley has control over and found that we could force change the password of the user damundsen** 

-  From here, we found that the **damundsen user can add a member to the Help Desk Level 1 group using GenericWrite privileges** 

- The **Help Desk Level 1 group is nested into the Information Technology group**, which grants members of that group any rights provisioned to the Information Technology group 


Now let's look around and see if members of Information Technology can do anything interesting. Once again, doing our search using Get-DomainObjectACL shows us that members of the Information Technology group have GenericAll rights over the user adunn, which means we could: 

- Modify group membership
- Force change a password
- Perform a targeted Kerberoasting attack and attempt to crack the user's password if it is weak 


#### Investigating the Information Technology Group 

```
PS C:\> $itgroupsid = Convert-NameToSid "Information Technology" 
PS C:\> Get-DomainObjectAcl -ResolveGUIDs -Identity * | ? {$_.SecurityIdentifier -eq $itgroupsid} -Verbose
```

![AD Enumeration](/ACL-Enumeration/images/itgroup.png) 

Finally, let's see if the **adunn** user has any type of interesting access that we may be able to leverage to get closer to our goal. 

![AD Enumeration](/ACL-Enumeration/images/adunn.png) 

The output above shows that our **adunn user has DS-Replication-Get-Changes and DS-Replication-Get-Changes-In-Filtered-Set rights over the domain object**. This means that this user can be leveraged to perform a **DCSync attack**. We will cover this attack in-depth in the DCSync section. 


![AD Enumeration](/ACL-Enumeration/images/forend-user.png) 

- The forend user has GenericAll write over the Dagmar Payne User.
