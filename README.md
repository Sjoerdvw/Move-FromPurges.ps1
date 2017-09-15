# Move-FromPurges.ps1
Purpose of this script is to be able to recover items from the hidden Purges subfolder in situations where a Retention policy or other 
has moved large amounts to it. At the moment it only recover email messages from the purges folder, but will have more capabilities in the future.

Requirements:
Without the following below, the script cannot run, there are no alternatives.
1.	You need the EWS Managed API located here: https://www.microsoft.com/en-us/download/details.aspx?id=42951 
2.	You need to install PowershellGet (OneGet) if you are running on Powershell Version older than 5 (the script does not need to be run from a server, recommended run it from Windows 10 as that is what it was tested on): https://github.com/OneGet/oneget 
3.	From an Elevated (Admin) powershell window:
a.	Install-Module PoshRsJob

4.	You need an EWS Impersonation Account, this account is a normal user, no mailbox required that is given special priveleges to impersonate users via EWS to do the work in the script. Instructions below:
https://msdn.microsoft.com/en-us/library/office/dn722376(v=exchg.150).aspx 
  
Steps for Purge Recovery Script:
1.	You need an export of the Primary SMTP address of the users you want to recover, this can be in a variable or txt file (no headers)
Ex:  
Get-Mailbox | select -ExpandProperty PrimarySmtpAddress | Out-File C:\Move-FromPurges.ps1\export-users.txt  

2.	Before running the script import the list into a variable so we can use it to check all mailboxes.  
 
3.	The script has many parameters, some required:  
a.	Required: -Mailboxes $variable <- This command is your imported data of SMTP addresses for users to recover  
b.	Required: -AccountWithImpersonationRights “UPN@contoso.com” <- Your EWS impersonation account configured in prereqs  
c.	Required: -StartDate “01-01-2015” <-The beginning date you want emails recovered from (specifying a large range will significantly  increase completion time)  
d.	Required: -EndDate “09-14-2017” <- The end date for the range you want emails recover from (specifying a large range will significantly increase completion time)  
e.	Optional: -subfolder “Name” <- By default all messages are recovered to inbox, you can specify a subfolder to recover messages to under the Inbox, it will create it if it does not exist  
f.	Optional: -whatif <-Runs the code but does not actually move any items, it will log what it would do  
g.	Optional: -PageLimit 500 <- EWS can only return up to 1000 items at a time, the default in the script is 100. Specifying a higher value than 1000 will have no effect.  
h.	Optional: -logpath “FolderPath” <- Specify a log path to store logs in, default is to store in the directory running script from. Just specify the folder.  
i.	Optional: -threadlimit 5 <- The maximum number of purge recover jobs to run at once. Specifying too high a number may cause throttling or even crash the script, 5 jobs at once is the default, don’t suggest going higher than 10  
   
4.	If you want to do a “Dry Run” you can run with the -WhatIf switch, an example is below:
.\Move-FromPurges.ps1 -Mailboxes $mbs -subfolder "Recovery" -AccountWithImpersonationRights "ews.impersonator@gamersbot.com" -startdate 01-01-2017 -enddate 09-14-2017 -whatif    
