## Enumerating AD Users


- When starting enumeration in an AD environment, arguably, the most important objects are domain users. Users have access to computers and are assigned permissions to perform a variety of functions throughout the domain. We need to control user accounts to move laterally and vertically within a network to reach the assessment goal. 


### Key AD User Data Points


- We can use **PowerView and SharpView** to enumerate a wealth of information about AD users. We can start by getting a count of how many users are in the target domain. We switch between the two tools frequently in this course because it is important to know them both. Sometimes you won't be able to run powershell commands and at other times you may not be able to run executable's. Any time you see SharpView usage, it should be possible to do it in PowerView by just removing SharpView.exe. SharpView does not have the latest PowerSploit features, so it may not be possible to run PowerView commands within SharpView. 


- Next, let's explore the **Get-DomainUser** function. If we provide the **-Help** flag to any `SharpView function, we can see all of the parameters that the function accepts.


	PS C:\htb> .\SharpView.exe Get-DomainUser -Help

	Get_DomainUser -Identity <String[]> -DistinguishedName <String[]> -SamAccountName <String[]> -Name <String[]> -MemberDistinguishedName <String[]> -MemberName <String[]> -SPN <Boolean> -AdminCount <Boolean> -AllowDelegation <Boolean> -DisalowDelegation <Boolean> -TrustedToAuth <Boolean> -PreauthNotRequired <Boolean> -KerberosPreauthNotRequired <Boolean> -Noreauth <Boolean> -Domain <String> -LDAPFilter <String> -Filter <String> -Properties <String[]> -SearchBase <String> -ADPath <String> -Server <String> -DomainController <String> -SearchScope <SearchScope> -ResultPageSize <Int32> -ServerTimLimit <Nullable`1> -SecurityMasks <Nullable`1> -Tombstone <Boolean> -FindOne <Boolean> -ReturnOne <Boolean> -Credential <NetworkCredential> -Raw <Boolean> -UACFilter <UACEnum>. 
	


![PowerView / SharpView](/Enumerating-AD-Users/images/domain-user.png) 


- Below are some of the most important properties to gather about domain users. Let's take a look at the harry.jones user. 


	PS C:\htb> Get-DomainUser -Identity harry.jones -Domain inlanefreight.local | Select-Object -Property name,samaccountname,description,memberof,whencreated,pwdlastset,lastlogontimestamp,accountexpires,admincount,userprincipalname,serviceprincipalname,mail,useraccountcontrol. 
	


![PowerView / SharpView](/Enumerating-AD-Users/images/domain-user-2.png) 


![PowerView / SharpView](/Enumerating-AD-Users/images/domain-user-3.png) 


- It is useful to enumerate these properties for ALL domain users and export them to a CSV file for offline processing. 


![PowerView / SharpView](/Enumerating-AD-Users/images/csv.png) 


	PS C:\htb> Get-DomainUser * -Domain inlanefreight.local | Select-Object -Property name,samaccountname,description,memberof,whencreated,pwdlastset,lastlogontimestamp,accountexpires,admincount,userprincipalname,serviceprincipalname,mail,useraccountcontrol | Export-Csv .\inlanefreight_users.csv -NoTypeInformation. 




- Once we have gathered information on all users, we can begin to perform more specific user enumeration by obtaining a list of users that do not require Kerberos pre-authentication and can be subjected to an ASREPRoast attack. 


	PS C:\htb> .\SharpView.exe Get-DomainUser -KerberosPreauthNotRequired -Properties samaccountname,useraccountcontrol,memberof 
	

	[Get-DomainSearcher] search base: LDAP://DC01.INLANEFREIGHT.LOCAL/DC=INLANEFREIGHT,DC=LOCAL
	[Get-DomainUser] Searching for user accounts that do not require kerberos preauthenticate
	[Get-DomainUser] filter string: (&(samAccountType=805306368)(userAccountControl:1.2.840.113556.1.4.803:=4194304))


![PowerView / SharpView](/Enumerating-AD-Users/images/kerberos.png) 


![PowerView / SharpView](/Enumerating-AD-Users/images/kerberos-2.png) 


### Kerberos Constrained and Unconstrained Delegation


- Let's also gather information about users with Kerberos constrained delegation. 


![PowerView / SharpView](/Enumerating-AD-Users/images/constrained.png) 


- While we're at it, we can look for users that allow unconstrained delegation. 


	PS C:\htb> .\SharpView.exe Get-DomainUser -LDAPFilter "(userAccountControl:1.2.840.113556.1.4.803:=524288)"


![PowerView / SharpView](/Enumerating-AD-Users/images/unconstrained.png) 


### Password in Description Field

- We can also check for any domain users with sensitive data such as a password stored in the description field. 


	PS C:\htb> Get-DomainUser -Properties samaccountname,description | Where {$_.description -ne $null} 
	


![PowerView / SharpView](/Enumerating-AD-Users/images/description.png) 


### Service Principal Name {SPNs} Kerberoasting Check


- Next, let's enumerate any users with Service Principal Names (SPNs) that could be subjected to a Kerberoasting attack. 


![PowerView / SharpView](/Enumerating-AD-Users/images/spns.png) 


	PS C:\htb> .\SharpView.exe Get-DomainUser -SPN -Properties samaccountname,memberof,serviceprincipalname
	


### Foreign Domain Membership Check


- Finally, we can enumerate any users from other (foreign) domains with group membership within any groups in our current domain. We can see that the user **harry.jones** from the **FREIGHTLOGISTICS.LOCAL domain** is in our current domain's **administrators group**. If we compromise the current domain, we may obtain credentials for this user from the NTDS database and authenticate into the **FREIGHTLOGISTICS.LOCAL domain**. 


![PowerView / SharpView](/Enumerating-AD-Users/images/foreign.png) 

![PowerView / SharpView](/Enumerating-AD-Users/images/foreign-2.png) 


### Active Directory Trust Check


- Another useful command is checking for **users with Service Principal Names (SPNs) set in other domains** that we can authenticate into via inbound or bi-directional trust relationships with forest-wide authentication allowing all users to authenticate across a trust or selective-authentication set up which allows specific users to authenticate. Here we can see one account in the **FREIGHTLOGISTICS.LOCAL domain**, which could be leveraged to Kerberoast across the forest trust. 


	PS C:\htb> Get-DomainUser -SPN -Domain freightlogistics.local | select samaccountname,memberof,serviceprincipalname | fl
	

![PowerView / SharpView](/Enumerating-AD-Users/images/trusts.png) 


### Password Set Times


- Analyzing the Password Set times is incredibly important when performing password sprays. Organizations are much more likely to find an automated password spray across all accounts than at a few guesses towards a small group of accounts. 


- If you see a **several passwords set at the same time**, this indicates they were set by the Help Desk and may be the same. Because of Password Lockout Policies, you may not be able to exceed four failed passwords in fifteen minutes. However, if you think the password is the same across 20 accounts, for one user, you can guess passwords along the line of "Password2020" for a different use, you can use the company name like "Freight2020!". 


- Additionally, if you see the password was set in July of 2019; then you can normally exclude "2020" from your password guessing and probably shouldn't guess variations that wouldn't make sense, such as "Winter2019." 


- If you see an old password that was set 2 years ago, chances are this password is weak and also one of the first accounts I would recommend guessing the password to before launching a large Password Spray. 


- In most organizations, administrators have multiple accounts. If you see the administrator changing his "user account" around the same time as his "Administrator Account", they are highly likely to use the same password for both accounts. 

- The following command will display all password set times.


![PowerView / SharpView](/Enumerating-AD-Users/images/password.png) 


	PS C:\htb> Get-DomainUser -Properties samaccountname,pwdlastset,lastlogon -Domain InlaneFreight.local | select samaccountname, pwdlastset, lastlogon | Sort-Object -Property pwdlastset
	


- If you want only to show passwords set before a certain date: 

	PS C:\htb> Get-DomainUser -Properties samaccountname,pwdlastset,lastlogon -Domain InlaneFreight.local | select samaccountname, pwdlastset, lastlogon | where { $_.pwdlastset -lt (Get-Date).addDays(-90) }
	


![PowerView / SharpView](/Enumerating-AD-Users/images/password-2.png) 
