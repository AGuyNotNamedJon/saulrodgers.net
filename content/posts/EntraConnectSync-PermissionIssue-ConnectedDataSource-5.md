+++

title = "Microsoft Entra ID Connect Sync - Connected data source error code 5"
description = ""
date = "2026-01-29"
preview = ""
draft = "false"
tags = [ "Entra ID", "Conenct Sync", "Connected data source error code: 5", "permission-issue", "Entra ID Connect", "permission issue", "Azure AD Connect", "Microsoft Entra Connect", "Active Directory" ]
categories = [ "Entra ID" ]
type = "posts"
series = []

+++

Today I had come across a new error I had not previously experienced where only some of the Microsoft 365 Groups were not writting back to our on-premises Active Directory anymore. This issue was the loverly "permission-issue" but with the Connected data source error code: "5".

While I have not been able to pinpoint the root cause of this error starting (will update if I find out), I was able to work out the solution.

To resolve this error you will need to open PowerShell as an administrator on the server running Entra ID Connect Sync and perfrom the following actions

Import the PowerShell module for Entra ID Connect Sync.

    Import-Module "C:\Program Files\Microsoft Azure Active Directory Connect\AdSyncConfig\AdSyncConfig.psm1"

Get the account name for the AD Connector. This account is most likely going to start with `MSOL_` unless you have chosen to customise this when setting it up.

    Get-ADSyncADConnectorAccount

Assign the Group Writeback Permissions. Replace `[ADConnectorAccountName]` with the `MSOL_` account name and replace `[ADDomain]` with the domain of the on-premises Active Directory (E.g. `CORP.EXAMPLE.com` or `EXAMPLE.LOCAL`).

    Set-ADSyncUnifiedGroupWritebackPermissions -ADConnectorAccountName "[ADConnectorAccountName]" -ADConnectorAccountDomain "[ADDomain]"

As a full example for step 3 assuming the Account Name was "MSOL_abc123def567" and the domain of active directory was "EXAMPLE.LOCAL".

    Set-ADSyncUnifiedGroupWritebackPermissions -ADConnectorAccountName "MSOL_abc123def567" -ADConnectorAccountDomain "EXAMPLE.LOCAL"
