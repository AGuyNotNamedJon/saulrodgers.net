+++

title = "Microsoft Entra ID Connect Sync - Connected data source error code 5"
description = ""
date = "2026-01-29"
preview = ""
draft = "true"
tags = [ "Entra ID", "Conenct Sync", "Connected data source error code: 5", "permission-issue", "Entra ID Connect", "permission issue", "Connected data source", "error code 5", "permission-issue 5", "Azure AD Connect", "Microsoft Entra Connect", "Microsoft 365", "PowerShell", "Active Directory" ]
categories = [ "Entra ID" ]
type = "posts"
series = []

+++

Today I had come across a new error I had not previously experienced where only some of the Microsoft 365 Groups were not writting back to our on-premises Active Directory anymore. This issue was the loverly "permission-issue" but with the Connected data source error code: "5".

While I have not been able to pinpoint the root cause of this error starting (will update if I find out), I was able to work out the solution.

To resolve this error you will need to open PowerShell as an administrator on the server running Entra ID Connect Sync and perfrom the following actions

1. Import the PowerShell module for Entra ID Connect Sync

    Import-Module "C:\Program Files\Microsoft Azure Active Directory Connect\AdSyncConfig\AdSyncConfig.psm1"

3. Get the account name for the AD Connector
    - This is most likely going to start with `MSOL_`

    Get-ADSyncADConnectorAccount

5. Assign the Group Writeback Permissions
    - Replace `[ADConnectorAccountName]` with the `MSOL_` account name in the previous step
    - Replace `[ADDomain]` with the domain of the on-premises Active Directory (E.g. `CORP.EXAMPLE.com` or `EXAMPLE.LOCAL`)

    Set-ADSyncUnifiedGroupWritebackPermissions -ADConnectorAccountName "[ADConnectorAccountName]" -ADConnectorAccountDomain "[ADDomain]"

As a full example for step 3 assuming the Account Name was "MSOL_abc123def567" and the domain of active directory was "EXAMPLE.LOCAL" you would type:

    Set-ADSyncUnifiedGroupWritebackPermissions -ADConnectorAccountName "MSOL_abc123def567" -ADConnectorAccountDomain "EXAMPLE.LOCAL"
