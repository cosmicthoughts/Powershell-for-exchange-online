# Essential PowerShell Commands for Exchange Online (Hybrid Environment)

A collection of useful PowerShell commands for managing Exchange Online in a hybrid setup. These commands help with mailbox management, delegation, forwarding, archiving, migrations, and more.
---
## Overview

This repository provides a set of practical PowerShell commands for managing Microsoft Exchange Online in a hybrid environment. It's designed for sysadmins and IT professionals who regularly work with mailbox reporting, delegation, and compliance-related tasks.

All commands are tested in hybrid configurations with directory sync (AAD Connect) enabled.

## Prerequisites

Before using these commands, ensure:

- You have the **Exchange Online PowerShell V2 (EXO V2) module** installed
- You are connected to Exchange Online PowerShell
- You have sufficient permissions (e.g. View-Only Recipients, Recipient Management, etc.)

### Connect to Exchange Online
```powershell
Connect-ExchangeOnline -UserPrincipalName youradmin@yourdomain.com
```

### Get a list of mailboxes and their primary email addresses
``` powershell
Get-Mailbox -ResultSize Unlimited | Select DisplayName,PrimarySMTPAddress
```

### Get all inactive mailboxes, and the aliases. Export to csv.
``` powershell
Get-Mailbox -InactiveMailboxOnly -ResultSize Unlimited | Select DisplayName, PrimarySMTPAddress, DistinguishedName, ExchangeGuid, WhenSoftDeleted, @{Name="Aliases";Expression={$_.EmailAddresses -match "^smtp:" -replace "smtp:" -join "; "}} | Export-Csv -Path "C:\Temp\InactiveMailboxes.csv" -NoTypeInformation -Encoding UTF8
```

### Find all shared mailboxes
``` powershell
Get-Mailbox -RecipientTypeDetails SharedMailbox -ResultSize Unlimited | Select DisplayName,PrimarySMTPAddress
```

### Check mailbox delegation (Full Access & Send As)
``` powershell
Get-Mailbox -ResultSize Unlimited | Select DisplayName,PrimarySMTPAddress, @{Name="FullAccess";Expression={(Get-MailboxPermission $_.Identity | Where-Object {($_.AccessRights -match "FullAccess") -and ($_.User -notmatch "NT AUTHORITY\\SELF")} | Select-Object User -ExpandProperty User) -join ", "}}, @{Name="SendAs";Expression={(Get-RecipientPermission $_.Identity | Where-Object {($_.AccessRights -match "SendAs")} | Select-Object Trustee -ExpandProperty Trustee) -join ", "}}
```

### List all mailboxes with forwarding enabled
``` powershell
Get-Mailbox -ResultSize Unlimited | Where-Object { $_.ForwardingSMTPAddress -or $_.ForwardingAddress } | Select DisplayName,PrimarySMTPAddress,ForwardingSMTPAddress,ForwardingAddress,DeliverToMailboxAndForward
```

### Get a list of all mail-enabled security groups
``` powershell
Get-Mailbox -Archive -ResultSize Unlimited | Select DisplayName,PrimarySMTPAddress,ArchiveStatus
```

### Find users with mailbox archive enabled
``` powershell
Get-Mailbox -Archive -ResultSize Unlimited | Select DisplayName,PrimarySMTPAddress,ArchiveStatus
```

### Check the last login time of a mailbox
``` powershell
Get-MailboxStatistics -Identity user@domain.com | Select DisplayName,LastLogonTime
```

### Get a list of transport rules (mail flow rules)
``` powershell
Get-TransportRule | Select Name,Mode,Priority,Comments
```

### Get the mailbox size of a user
``` powershell
Get-MailboxStatistics -Identity user@domain.com | Select DisplayName,TotalItemSize,ItemCount
```

### Find users with external email forwarding enabled
``` powershell
Get-Mailbox -ResultSize Unlimited | Where-Object { $_.ForwardingSMTPAddress -match "@" -and $_.ForwardingSMTPAddress -notmatch "yourdomain.com" } | Select DisplayName,PrimarySMTPAddress,ForwardingSMTPAddress
```

### List users with Send on Behalf permissions
``` powershell
Get-Mailbox -ResultSize Unlimited | Select DisplayName, GrantSendOnBehalfTo
```

### List mailboxes with litigation hold enabled
``` powershell
Get-Mailbox -ResultSize Unlimited | Where-Object { $_.LitigationHoldEnabled -eq $true } | Select DisplayName, LitigationHoldDate
```

### List mailboxes with litigation hold or in-place hold enabled
``` powershell
Get-Mailbox -ResultSize Unlimited | Where-Object { $_.LitigationHoldEnabled -or $_.InPlaceHolds.Count -gt 0 } | Select DisplayName,LitigationHoldEnabled,InPlaceHolds
```

### List mailboxes over a certain size
``` powershell
Get-Mailbox -ResultSize Unlimited | Get-MailboxStatistics | Where-Object { $_.TotalItemSize.Value.ToMB() -gt 5000 } | Select DisplayName,TotalItemSize
```

### List mailboxes with mailbox auditing disabled
``` powershell
Get-Mailbox -ResultSize Unlimited | Where-Object { $_.RecipientTypeDetails -eq "UserMailbox" -and $_.AuditEnabled -eq $false } | Select DisplayName,PrimarySMTPAddress
```

### List all transport rules
``` powershell
Get-TransportRule
```

### List all transport rules containing external domains
``` powershell
Get-TransportRule | Where-Object { $_.SentToScope -eq "NotInOrganization" -or $_.FromScope -eq "NotInOrganization" } | Select Name,Comments
```

### List all mailboxes and their quota settings.
``` powershell
Get-Mailbox -ResultSize Unlimited | Select DisplayName,ProhibitSendQuota,ProhibitSendReceiveQuota,IssueWarningQuota
```

### List all mailboxes over 90% of quota
``` powershell
Get-Mailbox -ResultSize Unlimited | Get-MailboxStatistics | Where-Object {
    $_.StorageLimitStatus -eq "IssueWarning" -or $_.StorageLimitStatus -eq "ProhibitSend"
} | Select DisplayName,TotalItemSize,StorageLimitStatus
```

### List all mailboxes with forwarding rules
``` powershell
Get-InboxRule -Mailbox user@domain.com | Where-Object { $_.ForwardTo -ne $null -or $_.RedirectTo -ne $null } | Select Name,ForwardTo,RedirectTo
```



## Contributing

Pull requests are welcome. If you have a useful snippet to share or corrections to suggest, feel free to contribute!
