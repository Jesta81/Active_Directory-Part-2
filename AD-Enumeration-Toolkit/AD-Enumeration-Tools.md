## Active Directory Enumeration Tools


- [Bloodhound](https://github.com/BloodHoundAD/BloodHound) Used to visually map out AD relationships and help plan attack paths that may otherwise go unnoticed. Uses the [SharpHound](https://github.com/BloodHoundAD/BloodHound/tree/master/Ingestors) PowerShell or C# ingestor to gather data to later be imported into the BloodHound JavaScript (Electron) application with a [Neo4j](https://github.com/BloodHoundAD/BloodHound/tree/master/Ingestors) database for graphical analysis of the AD environment. 


- [BloodHound.py](https://github.com/fox-it/BloodHound.py) A Python-based BloodHound ingestor based on the [Impacket toolkit](https://github.com/CoreSecurity/impacket/). It supports most BloodHound collection methods and can be run from a non-domain joined attack box. The output can be ingested into BloodHound 3.0 for analysis. 


- [PowerView](https://github.com/PowerShellMafia/PowerSploit/blob/master/Recon/PowerView.ps1) / [SharpView](https://github.com/dmchell/SharpView) A PowerShell tool and a **.NET** port of the same used to gain situational awareness in AD. These tools can be used as replacements for various Windows net* commands and more. PowerView and SharpView can help us gather much of the data that BloodHound does, but it requires more work to make meaningful relationships among all of the data points. These tools are great for checking what additional access we may have with a new set of credentials, targeting specific users or computers, or finding some "quick wins" such as users that can be attacked via Kerberoasting or ASREPRoasting. 


- [CrackMapExec](https://github.com/byt3bl33d3r/CrackMapExec) CME is an enumeration, attack, and post-exploitation toolkit which can help us greatly in enumeration and performing attacks with the data we gather. CME attempts to "live off the land" and abuse built-in AD features and protocols such as SMB, WMI, WinRM, and more. 


- [PingCastle](https://github.com/vletoux/pingcastle) Used for auditing the security level of an AD environment based on a risk assessment and maturity framework (based on [CMMI](https://en.wikipedia.org/wiki/Capability_Maturity_Model_Integration) adapted to AD security). 


- [PowerUpSQL](https://github.com/NetSPI/PowerUpSQL) This tool is used for SQL Server discovery, configuration auditing, privilege escalation, and post-exploitation. 


- [Snaffler](https://github.com/SnaffCon/Snaffler) Useful for finding information (such as credentials) in Active Directory on computers with accessible file shares. 


- [Group3r](https://github.com/Group3r/Group3r) Group3r is useful for auditing and finding security misconfigurations in AD Group Policy Objects (GPO). 


- [MailSniper](https://github.com/dafthack/MailSniper) A tool for searching through email inboxes in a Microsoft Exchange environment for specific keywords/terms that may be used to enumerate sensitive data (such as credentials) which could be used for lateral movement and privilege escalation. It can search a user's individual mailbox or by a user with Exchange Administrator privileges to enumerate all mailboxes in a domain. It can also be used for password spraying, enumerating domain users/domains, checking mailbox permissions, and gathering the Global Address List (GAL) from Outlook Web Access (OWA) and Exchange Web Services (EWS). 


- [windapsearch](https://github.com/ropnop/windapsearch) A tool used to extract various data from a target AD environment. The data can be output in Microsoft Excel format with summary views and analysis to assist with analysis and paint a picture of the environment's overall security state. 


- [Active Directory Explorer](https://docs.microsoft.com/en-us/sysinternals/downloads/adexplorer) Active Directory Explorer (AD Explorer) is an AD viewer and editor. It can be used to navigate an AD database and view object properties and attributes. It can also be used to save a snapshot of an AD database for off-line analysis. When an AD snapshot is loaded, it can be explored as a live version of the database. It can also be used to compare two AD database snapshots to see changes in objects, attributes, and security permissions. 
