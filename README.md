# Project 2: Automated JML Identity Lifecycle  Microsoft Entra ID

## Overview

Built a fully automated Joiner/Mover/Leaver (JML) identity lifecycle pipeline using Microsoft Entra ID, Azure Logic Apps, Azure Automation, and Microsoft Graph API. Started with manual provisioning to understand the fundamentals, then extended it into a fully event-driven automated offboarding pipeline replicating how enterprise IAM teams handle terminations at scale.

**Why it matters:** Orphaned accounts are consistently listed as a top attack vector in breach reports. This lab demonstrates end-to-end identity lifecycle management  not just provisioning, but automated, auditable offboarding triggered by an event, with no manual steps required.

---

## Tools & Technologies

- Microsoft Entra ID (P2)
- Microsoft Graph API
- PowerShell
- Azure Automation (Runbooks)
- Azure Logic Apps
- Managed Identity (no stored credentials)
- Entra ID Portal

---

## Part 1: Manual JML Lifecycle (Foundation)

### Joiner — New User Onboarding
- Created new user accounts in Entra ID with appropriate attributes
- Assigned users to applications and groups based on role
- Configured MFA enrollment requirement on first login
- Documented provisioning steps to simulate a repeatable onboarding workflow

### Mover — Role Change
- Modified group memberships and application assignments to reflect a department transfer
- Removed access to previous role's applications
- Updated user attributes to reflect new department and manager

### Leaver — Offboarding
- Disabled the user account immediately upon simulated termination
- Revoked active sessions and removed application assignments
- After hold period, permanently deleted the account from the directory
- Verified no orphaned access remained across assigned applications

### Key Concepts Practiced
- Joiner/Mover/Leaver (JML) lifecycle management
- Application assignment and role-based access
- Session revocation and account disabling
- Orphaned account prevention
- Access removal verification

---

## Part 2: Automated Pipeline (Event-Driven Offboarding)

Extended the manual JML lab with a fully event-driven pipeline. When an HTTP POST is sent simulating an HR termination event, the entire offboarding process runs automatically  no manual steps required.

### Pipeline Architecture

```
HTTP POST (HR Termination Event)
        ↓
   Azure Logic App
        ↓
Azure Automation Runbook
        ↓
  Microsoft Graph API
        ↓
 ┌──────────────────────┐
 │ Disable account      │
 │ Revoke sessions      │
 │ Remove groups        │
 │ Strip licenses       │
 └──────────────────────┘
        ↓
  Entra Audit Logs
        ↓
  Email Notification
```

### What Was Built

**Phase 1 — Test User**
- Created test user in Entra ID assigned to groups and apps for offboarding simulation

**Phase 2 — Azure Automation Account**
- Created Automation Account with System-Assigned Managed Identity
- Managed Identity authenticates to Graph API — no stored credentials

**Phase 3 — Graph API Permissions**
- Granted Managed Identity: User.ReadWrite.All, GroupMember.ReadWrite.All, Directory.ReadWrite.All
- Configured via PowerShell using Cloud Shell

**Phase 4 — PowerShell Runbook**
- Runbook: `Invoke-JMLOffboard`
- Accepts UserPrincipalName as parameter
- Authenticates via Managed Identity
- Calls Graph API to:
  - Disable account (`accountEnabled: false`)
  - Revoke all active sign-in sessions
  - Remove all group memberships
  - Strip all license assignments
- Logs each step to output

**Phase 5 — Logic App Trigger**
- Logic App: `jml-trigger` (Consumption tier)
- Trigger: When HTTP request is received
- Accepts JSON payload: `{ "userPrincipalName": "user@tenant.onmicrosoft.com", "action": "terminate" }`
- Fires Azure Automation runbook automatically
- Sends email notification on completion

**Phase 6 — Testing**
- Fired trigger via HTTP POST
- Verified: Logic App runs history showed green checkmarks
- Verified: Runbook output showed each step completed
- Verified: User account disabled in Entra ID
- Verified: Group memberships removed
- Verified: Audit logs captured all actions
- Verified: Email notification received

### Key Results
- Full offboarding from trigger to completion: **under 5 minutes**
- Zero manual steps after trigger fired
- No stored credentials — Managed Identity handles all authentication
- Full audit trail in Entra ID audit logs
- Scales to production by swapping HTTP trigger for Workday/SAP webhook

---

## PowerShell Runbook Script

```powershell
param(
  [Parameter(Mandatory=$true)]
  [string]$UserPrincipalName
)

# Authenticate using managed identity
Connect-AzAccount -Identity | Out-Null
$token = (Get-AzAccessToken -ResourceUrl "https://graph.microsoft.com").Token
$headers = @{ Authorization = "Bearer $token"; "Content-Type" = "application/json" }
$graphBase = "https://graph.microsoft.com/v1.0"

# Get user ID from UPN
Write-Output "Looking up user: $UserPrincipalName"
$user = Invoke-RestMethod -Uri "$graphBase/users/$UserPrincipalName" -Headers $headers
$userId = $user.id
Write-Output "Found user ID: $userId"

# Step 1: Disable account
Write-Output "Disabling account..."
Invoke-RestMethod -Method PATCH -Uri "$graphBase/users/$userId" `
  -Headers $headers -Body '{"accountEnabled": false}'

# Step 2: Revoke all active sessions
Write-Output "Revoking sessions..."
Invoke-RestMethod -Method POST `
  -Uri "$graphBase/users/$userId/revokeSignInSessions" -Headers $headers

# Step 3: Remove all group memberships
Write-Output "Removing group memberships..."
$groups = Invoke-RestMethod -Uri "$graphBase/users/$userId/memberOf" -Headers $headers
foreach ($group in $groups.value) {
  try {
    Invoke-RestMethod -Method DELETE `
      -Uri "$graphBase/groups/$($group.id)/members/$userId/`$ref" -Headers $headers
    Write-Output "Removed from group: $($group.displayName)"
  } catch {
    Write-Output "Could not remove from $($group.displayName): $_"
  }
}

# Step 4: Remove license assignments
Write-Output "Removing licenses..."
$licenses = Invoke-RestMethod -Uri "$graphBase/users/$userId/licenseDetails" -Headers $headers
$skuIds = $licenses.value | ForEach-Object { $_.skuId }
if ($skuIds.Count -gt 0) {
  $body = @{ addLicenses = @(); removeLicenses = $skuIds } | ConvertTo-Json
  Invoke-RestMethod -Method POST `
    -Uri "$graphBase/users/$userId/assignLicense" -Headers $headers -Body $body
  Write-Output "Licenses removed"
} else {
  Write-Output "No licenses to remove"
}

Write-Output "Offboarding complete for $UserPrincipalName"
```

---

## Screenshots
## Screenshots

![Job Output](screenshots/last_auto.png)

![Logic App Trigger](screenshots/automated12.png)

![PowerShell Runbook](screenshots/automation8.png)

![Graph API Permissions](screenshots/automation5.png)
---

## Key Concepts Demonstrated
- Event-driven IAM automation
- Managed Identity authentication (no stored credentials)
- Microsoft Graph API for identity management
- Azure Logic Apps as orchestration layer
- Azure Automation for PowerShell execution
- Least privilege  Managed Identity granted minimum required permissions
- Audit trail and compliance logging
- Scalable architecture HTTP trigger swappable for Workday/SAP webhook in production

---

*Built by Deniqua Collins | github.com/Deniqua20 | IAM Engineer in progress 💪*
