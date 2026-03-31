[README (1).md](https://github.com/user-attachments/files/26390560/README.1.md)
# Project 1: Zero Trust Conditional Access Lab

## Overview
This project demonstrates design and implementation of a production-grade Conditional Access policy framework in Microsoft Entra ID, aligned with Zero Trust principles and NIST 800-53 controls.

## Prerequisites
- Microsoft Entra ID tenant (Free trial: https://entra.microsoft.com)
- Global Administrator or Conditional Access Administrator role
- At least 2 test user accounts (one licensed, one unlicensed)
- Optional: Entra ID P1 license (free 30-day trial available)

## Policies Implemented

### Policy 1: Require MFA for All Users — All Cloud Apps
**Purpose:** Baseline MFA enforcement  
**Configuration:**
- Users: All users
- Cloud apps: All cloud apps
- Conditions: None (catches everything)
- Grant: Require multifactor authentication
- **NIST 800-53 mapping:** IA-2(1), IA-2(2)

### Policy 2: Block Legacy Authentication
**Purpose:** Eliminate credential stuffing attack surface via legacy protocols (IMAP, POP3, SMTP AUTH, Basic Auth)  
**Configuration:**
- Users: All users
- Cloud apps: All cloud apps
- Conditions: Client apps = Exchange ActiveSync clients, Other clients
- Grant: Block access
- **NIST 800-53 mapping:** AC-17, IA-8

### Policy 3: Require Compliant Device for Sensitive Apps
**Purpose:** Ensure only managed devices access high-value applications  
**Configuration:**
- Users: All users
- Cloud apps: [Target your most sensitive apps]
- Conditions: None
- Grant: Require device to be marked as compliant (Intune)
- **NIST 800-53 mapping:** AC-19, CM-6

### Policy 4: Restrict Access to Named Locations (U.S. Only)
**Purpose:** Block sign-ins from unexpected geographies  
**Configuration:**
- Users: All users
- Cloud apps: All cloud apps
- Conditions: Locations = All locations EXCEPT "United States (Named Location)"
- Grant: Block access
- **NIST 800-53 mapping:** AC-17(1)

### Policy 5: Risk-Based Adaptive MFA
**Purpose:** Dynamically escalate authentication requirements based on Entra ID Identity Protection risk signals  
**Configuration (requires P2):**
- Users: All users
- Cloud apps: All cloud apps
- Conditions: Sign-in risk = Medium, High
  - Medium → Require MFA
  - High → Block access
- **NIST 800-53 mapping:** IA-2(12), SI-4

### Policy 6: Phishing-Resistant MFA for Administrators
**Purpose:** All privileged roles must use FIDO2 or WHfB — no SMS/phone  
**Configuration:**
- Users: Directory roles = Global Admin, Security Admin, User Admin, Privileged Role Admin
- Cloud apps: All cloud apps
- Grant: Require authentication strength = Phishing-resistant MFA
- **NIST 800-53 mapping:** IA-2(1), AC-2(7)

## Testing Methodology

### Test 1: MFA Enforcement
1. Sign in as test user from browser
2. Confirm MFA prompt appears
3. Check Sign-in logs: Conditional Access tab → Policy applied = Policy 1

### Test 2: Legacy Auth Block
1. Attempt IMAP connection to Exchange Online using basic credentials
2. Confirm 403/block response
3. Verify in Sign-in logs: Client app = "Other clients", Status = Failure

### Test 3: Risk Policy (Simulated)
1. Use Entra Identity Protection > Risk detections > Confirm compromised (or use anonymous browser)
2. Trigger risky sign-in
3. Confirm MFA prompt (Medium) or block (High)

## Lessons Learned / Design Decisions

**Why "All users" instead of targeting specific groups?**  
Starting with broad application and using exclusion groups is safer — you won't accidentally miss a user population.

**Why block legacy auth before requiring compliant device?**  
Legacy auth clients can't satisfy device compliance. Blocking them first prevents a bypass path.

**Why separate policy for admins?**  
Admin accounts are highest risk. They need stronger authentication guarantees even if the org-wide policy only requires regular MFA.

## Screenshots
*Add your own screenshots to the `/screenshots` folder:*
- `sign-in-log-mfa-enforced.png`
- `ca-policy-list.png`
- `legacy-auth-block-log.png`
- `risk-policy-mfa-trigger.png`

## Relevant SC-300 Exam Objectives
- Implement and manage Conditional Access policies (SC-300: 20–25%)
- Implement authentication and access management
- Plan and implement identity protection

## References
- [Microsoft: Zero Trust with Conditional Access](https://learn.microsoft.com/en-us/entra/identity/conditional-access/plan-conditional-access)
- [NIST SP 800-207: Zero Trust Architecture](https://csrc.nist.gov/publications/detail/sp/800-207/final)
- [NIST SP 800-53 Rev. 5: IA Family](https://csrc.nist.gov/publications/detail/sp/800-53/rev-5/final)
