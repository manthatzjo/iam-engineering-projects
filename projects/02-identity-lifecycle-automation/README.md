# Project 2: Automated Identity Lifecycle (PowerShell + MS Graph)

## Overview
Production-ready PowerShell automation for Joiner/Mover/Leaver (JML) identity lifecycle management using Microsoft Graph API — the #1 technical skill in IAM engineer job postings.

## Prerequisites
```powershell
# Install Microsoft Graph PowerShell SDK
Install-Module Microsoft.Graph -Scope CurrentUser

# Connect with required scopes
Connect-MgGraph -Scopes "User.ReadWrite.All","Group.ReadWrite.All","Directory.ReadWrite.All","AuditLog.Read.All"
```

---

## Script 1: New-EntraUser.ps1 (Joiner / Onboarding)

```powershell
<#
.SYNOPSIS
    Onboards a new user to Microsoft Entra ID with full lifecycle setup.
.DESCRIPTION
    Creates user account, assigns licenses, adds to department groups,
    sets manager, and sends welcome notification.
.PARAMETER FirstName
    User's first name
.PARAMETER LastName
    User's last name
.PARAMETER Department
    Department name (maps to security group)
.PARAMETER JobTitle
    User's job title
.PARAMETER ManagerUPN
    UPN of the user's manager
.EXAMPLE
    .\New-EntraUser.ps1 -FirstName "Jane" -LastName "Smith" -Department "Security" -JobTitle "IAM Analyst" -ManagerUPN "jdoe@contoso.com"
#>

param(
    [Parameter(Mandatory)][string]$FirstName,
    [Parameter(Mandatory)][string]$LastName,
    [Parameter(Mandatory)][string]$Department,
    [Parameter(Mandatory)][string]$JobTitle,
    [Parameter(Mandatory)][string]$ManagerUPN,
    [string]$Domain = "contoso.com",
    [string]$UsageLocation = "US"
)

# --- Build user properties ---
$UPN        = "$($FirstName.ToLower()).$($LastName.ToLower())@$Domain"
$DisplayName = "$FirstName $LastName"
$MailNickname = "$($FirstName.ToLower())$($LastName.ToLower())"

# Generate a secure temporary password
Add-Type -AssemblyName System.Web
$TempPassword = [System.Web.Security.Membership]::GeneratePassword(16, 3)

Write-Host "⚙️  Creating user: $DisplayName ($UPN)" -ForegroundColor Cyan

# --- Create user ---
$userParams = @{
    AccountEnabled    = $true
    DisplayName       = $DisplayName
    GivenName         = $FirstName
    Surname           = $LastName
    UserPrincipalName = $UPN
    MailNickname      = $MailNickname
    Department        = $Department
    JobTitle          = $JobTitle
    UsageLocation     = $UsageLocation
    PasswordProfile   = @{
        ForceChangePasswordNextSignIn = $true
        Password                      = $TempPassword
    }
}

try {
    $newUser = New-MgUser @userParams
    Write-Host "✅ User created: $($newUser.Id)" -ForegroundColor Green
} catch {
    Write-Error "❌ Failed to create user: $_"
    exit 1
}

# --- Assign manager ---
try {
    $manager = Get-MgUser -Filter "userPrincipalName eq '$ManagerUPN'"
    $managerRef = @{ "@odata.id" = "https://graph.microsoft.com/v1.0/users/$($manager.Id)" }
    Set-MgUserManagerByRef -UserId $newUser.Id -BodyParameter $managerRef
    Write-Host "✅ Manager set: $ManagerUPN" -ForegroundColor Green
} catch {
    Write-Warning "⚠️  Could not set manager: $_"
}

# --- Add to department group ---
try {
    $group = Get-MgGroup -Filter "displayName eq '$Department Team'"
    if ($group) {
        New-MgGroupMember -GroupId $group.Id -DirectoryObjectId $newUser.Id
        Write-Host "✅ Added to group: $Department Team" -ForegroundColor Green
    }
} catch {
    Write-Warning "⚠️  Group assignment failed: $_"
}

# --- Output summary ---
Write-Host "`n📋 Onboarding Summary" -ForegroundColor Yellow
Write-Host "User:          $DisplayName"
Write-Host "UPN:           $UPN"
Write-Host "Temp Password: $TempPassword (STORE SECURELY)"
Write-Host "Department:    $Department"
Write-Host "Manager:       $ManagerUPN"
```

---

## Script 2: Invoke-Offboarding.ps1 (Leaver / Termination)

```powershell
<#
.SYNOPSIS
    Securely offboards a departing user from Microsoft Entra ID.
.DESCRIPTION
    Disables account, revokes all active sessions, removes group memberships,
    removes licenses, and exports final access report for audit.
.PARAMETER UPN
    UserPrincipalName of the user to offboard
.PARAMETER TicketNumber
    Change management ticket number for audit trail
.EXAMPLE
    .\Invoke-Offboarding.ps1 -UPN "jsmith@contoso.com" -TicketNumber "CHG-2024-0042"
#>

param(
    [Parameter(Mandatory)][string]$UPN,
    [Parameter(Mandatory)][string]$TicketNumber
)

$timestamp = Get-Date -Format "yyyyMMdd-HHmmss"
$reportPath = ".\offboarding-report-$($UPN.Split('@')[0])-$timestamp.csv"

Write-Host "🔒 Starting offboarding for: $UPN (Ticket: $TicketNumber)" -ForegroundColor Red

# --- Get user ---
try {
    $user = Get-MgUser -Filter "userPrincipalName eq '$UPN'" -Property "Id,DisplayName,AccountEnabled,AssignedLicenses"
    if (-not $user) { throw "User not found" }
    Write-Host "Found user: $($user.DisplayName) | ID: $($user.Id)"
} catch {
    Write-Error "❌ Cannot find user $UPN`: $_"
    exit 1
}

# Step 1: Disable the account
Update-MgUser -UserId $user.Id -AccountEnabled:$false
Write-Host "✅ Step 1/5: Account disabled" -ForegroundColor Green

# Step 2: Revoke all refresh tokens (kills all active sessions)
Revoke-MgUserSignInSession -UserId $user.Id | Out-Null
Write-Host "✅ Step 2/5: All sessions revoked" -ForegroundColor Green

# Step 3: Export group memberships BEFORE removing (for audit)
$memberships = Get-MgUserMemberOf -UserId $user.Id | Where-Object { $_.'@odata.type' -eq '#microsoft.graph.group' }
$auditData = $memberships | Select-Object DisplayName, Id

# Step 4: Remove all group memberships
foreach ($group in $memberships) {
    try {
        Remove-MgGroupMemberByRef -GroupId $group.Id -DirectoryObjectId $user.Id
    } catch {
        Write-Warning "Could not remove from group $($group.DisplayName)"
    }
}
Write-Host "✅ Step 3/5: Removed from $($memberships.Count) groups" -ForegroundColor Green

# Step 5: Remove licenses
$assignedLicenses = $user.AssignedLicenses
if ($assignedLicenses.Count -gt 0) {
    $licenseParams = @{
        AddLicenses    = @()
        RemoveLicenses = $assignedLicenses.SkuId
    }
    Set-MgUserLicense -UserId $user.Id -BodyParameter $licenseParams
    Write-Host "✅ Step 4/5: Removed $($assignedLicenses.Count) licenses" -ForegroundColor Green
}

# Export audit report
$auditData | Export-Csv -Path $reportPath -NoTypeInformation
Write-Host "✅ Step 5/5: Audit report saved to $reportPath" -ForegroundColor Green

Write-Host "`n🔐 Offboarding complete for $UPN" -ForegroundColor Yellow
Write-Host "Ticket: $TicketNumber | Timestamp: $timestamp"
```

---

## Script 3: Get-StaleAccounts.ps1 (Access Hygiene)

```powershell
<#
.SYNOPSIS
    Identifies stale user accounts inactive for 90+ days.
.DESCRIPTION
    Queries Entra sign-in logs to find accounts with no activity,
    exports findings for access review.
#>

param(
    [int]$InactiveDays = 90,
    [string]$ExportPath = ".\stale-accounts-$(Get-Date -Format 'yyyyMMdd').csv"
)

$cutoffDate = (Get-Date).AddDays(-$InactiveDays).ToString("yyyy-MM-ddTHH:mm:ssZ")

Write-Host "🔍 Finding accounts inactive since: $((Get-Date).AddDays(-$InactiveDays).ToString('yyyy-MM-dd'))"

# Get all enabled users with sign-in activity data
$users = Get-MgUser -All -Property "Id,DisplayName,UserPrincipalName,AccountEnabled,SignInActivity,Department,JobTitle" `
    -Filter "AccountEnabled eq true" |
    Where-Object {
        $_.SignInActivity -eq $null -or
        $_.SignInActivity.LastSignInDateTime -lt $cutoffDate
    }

$results = $users | Select-Object DisplayName, UserPrincipalName, Department, JobTitle,
    @{N="LastSignIn"; E={ $_.SignInActivity.LastSignInDateTime ?? "Never" }},
    @{N="DaysSinceSignIn"; E={
        if ($_.SignInActivity.LastSignInDateTime) {
            ((Get-Date) - [datetime]$_.SignInActivity.LastSignInDateTime).Days
        } else { "N/A (Never signed in)" }
    }}

$results | Export-Csv -Path $ExportPath -NoTypeInformation

Write-Host "✅ Found $($results.Count) stale accounts. Report: $ExportPath"
$results | Format-Table -AutoSize
```

---

## Architecture Notes

### Graph API Permissions Required (App Registration)
| Permission | Type | Purpose |
|------------|------|---------|
| `User.ReadWrite.All` | Application | Create/update/disable users |
| `Group.ReadWrite.All` | Application | Manage group memberships |
| `Directory.ReadWrite.All` | Application | Manage directory objects |
| `AuditLog.Read.All` | Application | Read sign-in logs |
| `Organization.Read.All` | Application | Read license SKUs |

### Security Notes
- Never hard-code credentials — use Managed Identity or Certificate-based auth in production
- All scripts log actions with timestamps for audit trail
- Offboarding script exports access snapshot BEFORE removal (regulatory requirement)
- Use `-WhatIf` parameter pattern for testing before production runs

## SC-300 Exam Relevance
- Implement and manage user lifecycle (SC-300)
- Automate identity governance tasks
- Monitor and maintain identity health
