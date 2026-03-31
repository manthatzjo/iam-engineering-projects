[README.md](https://github.com/user-attachments/files/26390444/README.md)
# 🔐 Jonathan Dada — IAM / Entra ID Engineering Portfolio

> **IAM Professional | EY @ DHS | SC-300 Candidate | Security+ Certified**  
> Severn, MD | [LinkedIn](https://www.linkedin.com/in/jonathan-dada123) | jdada2003@gmail.com

---

## About This Portfolio

This repository showcases hands-on Identity & Access Management (IAM) engineering projects built in **Microsoft Entra ID (Azure AD)**, aligned with real-world enterprise job requirements from companies like Charles Schwab, BlackRock, and federal contractors. Each project demonstrates skills directly mapped to IAM engineer job postings across LinkedIn, Glassdoor, and Dice.

**Current Role:** Information Business Analyst (ICAM/IAM Operations) — Ernst & Young, supporting DHS  
**Studying:** SC-300 (Microsoft Identity and Access Administrator Associate)

---

## 📁 Project Index

| # | Project | Key Skills | Difficulty | Job Relevance |
|---|---------|-----------|------------|---------------|
| 1 | [Zero Trust Conditional Access Lab](#project-1) | Conditional Access, MFA, Named Locations | ⭐⭐ | ★★★★★ |
| 2 | [Automated Identity Lifecycle with PowerShell + MS Graph](#project-2) | PowerShell, MS Graph API, Automation | ⭐⭐⭐ | ★★★★★ |
| 3 | [Privileged Identity Management (PIM) Implementation](#project-3) | PIM, JIT Access, Least Privilege | ⭐⭐ | ★★★★★ |
| 4 | [Entra ID Governance — Access Reviews & Entitlement Management](#project-4) | IGA, Access Reviews, Entitlement Mgmt | ⭐⭐⭐ | ★★★★☆ |
| 5 | [SCIM Provisioning & SSO App Integration](#project-5) | SCIM, SAML, OIDC, SSO | ⭐⭐⭐ | ★★★★☆ |
| 6 | [Entra ID Security Monitoring Dashboard (KQL + Sentinel)](#project-6) | KQL, Sentinel, Identity Protection | ⭐⭐⭐ | ★★★★☆ |
| 7 | [Passwordless Authentication Deployment](#project-7) | FIDO2, WHfB, Authenticator App | ⭐⭐ | ★★★★☆ |
| 8 | [RBAC & Custom Role Engineering](#project-8) | RBAC, Custom Roles, Least Privilege | ⭐⭐ | ★★★★★ |

---

## Projects

---

### Project 1
## 🛡️ Zero Trust Conditional Access Lab
**`/projects/01-conditional-access-zero-trust/`**

**What it is:** Design and document a full Zero Trust Conditional Access policy set for a simulated enterprise using Entra ID's free/trial tenant.

**Policies built:**
- Require MFA for all cloud app access
- Block legacy authentication protocols
- Require compliant device for sensitive app access
- Named Location policy (restrict access outside U.S.)
- Sign-in risk-based policy (block High risk, require MFA for Medium)
- Admin roles require phishing-resistant MFA

**Deliverables in this folder:**
- `policy-designs/` — JSON exports of each Conditional Access policy
- `screenshots/` — Before/after sign-in logs showing policy enforcement
- `README.md` — Architecture decision rationale, NIST 800-53 control mapping
- `test-scenarios.md` — How each policy was validated

**Why it matters for hiring:**  
Every IAM engineer job posting lists Conditional Access as a must-have. This project shows you can *design* policies with business + security rationale, not just click through a UI.

**Resume/LinkedIn bullet:**
> *Designed and implemented Zero Trust Conditional Access policy framework in Microsoft Entra ID, covering MFA enforcement, risk-based sign-in policies, and legacy auth blocking — mapped to NIST 800-53 AC and IA control families.*

---

### Project 2
## ⚙️ Automated Identity Lifecycle with PowerShell + MS Graph API
**`/projects/02-identity-lifecycle-automation/`**

**What it is:** Build a PowerShell automation suite that handles joiner/mover/leaver (JML) workflows using Microsoft Graph API — the #1 technical skill employers look for.

**Scripts built:**
- `New-EntraUser.ps1` — Onboard new user: create account, assign licenses, add to groups, set manager
- `Update-UserProfile.ps1` — Mover workflow: update department, reassign group memberships, notify manager
- `Invoke-Offboarding.ps1` — Leaver workflow: disable account, revoke sessions, remove licenses, export access report
- `Get-StaleAccounts.ps1` — Find accounts inactive 90+ days for review
- `Export-AccessReport.ps1` — Generate CSV of all user role assignments for audit

**Deliverables:**
- All `.ps1` scripts with inline documentation
- `sample-output/` — CSV and JSON output examples
- `README.md` — Architecture, prerequisites, Graph API permissions required
- `graph-permissions.md` — Principle of least privilege app registration setup

**Why it matters:**  
PowerShell + MS Graph is the most cited technical skill in IAM job postings. Automation of JML is the core function of IGA platforms (SailPoint, Saviynt). This proves you can do it natively.

**Resume/LinkedIn bullet:**
> *Developed PowerShell automation suite leveraging Microsoft Graph API to streamline joiner/mover/leaver identity lifecycle workflows, reducing manual provisioning effort and enforcing least-privilege access controls.*

---

### Project 3
## 🔑 Privileged Identity Management (PIM) Implementation
**`/projects/03-pim-privileged-access/`**

**What it is:** Configure and document a full PIM deployment for a simulated org — Just-in-Time privileged access, approval workflows, and access reviews for admin roles.

**Configurations built:**
- JIT activation for Global Admin, Security Admin, User Admin roles
- Approval workflow requiring manager sign-off for privileged role activation
- Time-bound access (max 1 hour) with business justification requirement
- Alert configuration for suspicious PIM activity
- PIM access review scheduled monthly for all privileged roles
- Emergency access ("break glass") account documentation

**Deliverables:**
- `pim-config/` — Role settings screenshots and exported configs
- `access-review-results/` — Sample completed access review documentation
- `runbook.md` — Step-by-step operational runbook for PIM administration
- `README.md` — Design rationale and compliance mapping (NIST 800-53 AC-2, AC-6)

**Why it matters:**  
PIM appears in nearly every senior IAM job description. The SC-300 exam covers it heavily. This is one of the highest-value skills you can demonstrate.

**Resume/LinkedIn bullet:**
> *Implemented Privileged Identity Management (PIM) in Microsoft Entra ID with Just-in-Time access controls, approval workflows, and monthly access reviews for all privileged roles — reducing standing privileged access by 100%.*

---

### Project 4
## 📋 Entra ID Governance — Access Reviews & Entitlement Management
**`/projects/04-identity-governance/`**

**What it is:** Build and document an Identity Governance program using Entra ID Governance features — Access Reviews, Entitlement Management (access packages), and lifecycle workflows.

**Configurations built:**
- Access package for "New Hire Onboarding" — automatically assigns groups, SharePoint, and app access
- Recurring quarterly access review for all security group memberships
- Lifecycle workflow: auto-disable account 1 day after HR termination date
- Access package for "External Vendor" — time-limited guest access with auto-expiry
- Separation of Duties policy preventing conflicting role assignments

**Deliverables:**
- `access-packages/` — Configuration documentation and screenshots
- `lifecycle-workflows/` — Workflow logic diagrams and JSON definitions
- `access-review-cadence.md` — Review schedule and owner assignment plan
- `README.md` — IGA program overview, business justification

**Why it matters:**  
IGA (Identity Governance and Administration) is explicitly listed in high-paying IAM roles. This bridges your current operational work at DHS to the engineering/design skills employers want next.

**Resume/LinkedIn bullet:**
> *Designed and deployed Identity Governance program in Microsoft Entra ID Governance, including entitlement management access packages, automated lifecycle workflows, and quarterly access certifications.*

---

### Project 5
## 🔗 SCIM Provisioning & SSO App Integration
**`/projects/05-sso-scim-app-integration/`**

**What it is:** Integrate a real app (use a free tier app like Salesforce Dev, ServiceNow PDI, or AWS SSO) with Entra ID for automated SCIM provisioning and SSO via SAML/OIDC.

**Integrations built:**
- SAML 2.0 SSO integration (use AWS IAM Identity Center or a gallery app)
- OIDC/OAuth2 app registration with proper scopes and consent
- SCIM automatic user provisioning — users created in Entra auto-provisioned to target app
- Attribute mapping configuration (UPN → email, department → group)
- Provisioning logs review and error remediation

**Deliverables:**
- `saml-config/` — Federation metadata, attribute mapping screenshots
- `app-registration/` — OAuth2 app manifest and permission documentation  
- `provisioning-logs/` — Sample provisioning success/failure log analysis
- `README.md` — Step-by-step integration guide, troubleshooting tips

**Why it matters:**  
"SCIM provisioning," "SSO integration," "SAML/OIDC" appear in virtually every IAM engineer job. This is the skill that separates IAM operators from IAM engineers.

**Resume/LinkedIn bullet:**
> *Architected and implemented SCIM-based automated user provisioning and SAML/OIDC SSO integrations between Microsoft Entra ID and enterprise SaaS applications, enabling real-time identity synchronization and eliminating manual account creation.*

---

### Project 6
## 📊 Entra ID Security Monitoring with KQL + Microsoft Sentinel
**`/projects/06-identity-security-monitoring/`**

**What it is:** Build KQL queries and a Sentinel workbook to monitor identity threats — impossible travel, MFA fatigue attacks, PIM elevation spikes, and stale accounts.

**Queries & detections built:**
- Impossible travel detection (sign-ins from 2 countries within 1 hour)
- MFA fatigue/push bombing detection (10+ MFA prompts in 10 minutes)
- Privileged role assignment outside of PIM (shadow admin detection)
- Guest account sign-in anomaly detection
- Service principal credential expiry monitoring
- Failed sign-in spike by application

**Deliverables:**
- `kql-queries/` — All `.kql` files, one per detection use case
- `sentinel-workbook/` — JSON workbook template for identity dashboard
- `alert-rules/` — Scheduled alert rule configurations
- `README.md` — Threat model, detection rationale, MITRE ATT&CK mapping (T1078, T1556)

**Why it matters:**  
Security operations integration with IAM is a growing requirement. Shows you understand identity threats, not just configurations — making you valuable to both IAM and SOC teams.

**Resume/LinkedIn bullet:**
> *Developed Microsoft Sentinel KQL detection rules and identity security monitoring workbook covering MFA fatigue, impossible travel, and shadow admin detection — mapped to MITRE ATT&CK identity techniques.*

---

### Project 7
## 🔓 Passwordless Authentication Deployment
**`/projects/07-passwordless-authentication/`**

**What it is:** Design and document a full passwordless authentication rollout plan using FIDO2 security keys, Windows Hello for Business, and the Microsoft Authenticator app.

**Work products built:**
- Phased rollout plan (pilot → department → org-wide)
- FIDO2 security key registration and policy enforcement documentation
- Microsoft Authenticator number matching + additional context configuration
- Temporary Access Pass (TAP) policy for new hire bootstrapping
- User communication templates (email announcements, help desk FAQ)
- Authentication methods policy JSON export

**Deliverables:**
- `rollout-plan.md` — Full phased deployment plan with rollback criteria
- `auth-methods-policy/` — JSON exports of authentication method configurations
- `user-comms/` — Template emails and FAQ for end users
- `README.md` — Business case, security benefits, NIST AAL2/AAL3 alignment

**Why it matters:**  
Passwordless is one of the hottest IAM topics in 2025-2026. "Passwordless strategy" and "FIDO2" appear in senior IAM role descriptions at top firms. This shows forward-looking expertise.

**Resume/LinkedIn bullet:**
> *Designed enterprise passwordless authentication deployment strategy using FIDO2 security keys, Windows Hello for Business, and Microsoft Authenticator — aligned with NIST SP 800-63B AAL2/AAL3 requirements.*

---

### Project 8
## 🏗️ RBAC Engineering & Custom Role Design
**`/projects/08-rbac-custom-roles/`**

**What it is:** Design a custom RBAC model for a simulated organization, create custom Entra ID roles using PowerShell, and document the least-privilege access model.

**Work products built:**
- Custom "Help Desk Operator" role — only reset passwords and manage MFA, no admin panel access
- Custom "App Registration Owner" role — manage only assigned app registrations
- Custom "Security Reader +" role — read security data + export audit logs, nothing more
- Administrative Unit (AU) scoping — Help Desk can only manage users in their region/AU
- Role assignment matrix (RACI-style) documenting who has what and why

**Deliverables:**
- `custom-role-definitions/` — JSON role definition files for all custom roles
- `au-configuration/` — Administrative Unit setup documentation
- `rbac-matrix.xlsx` — Role assignment matrix spreadsheet
- `README.md` — Design rationale, least-privilege principles, audit trail

**Why it matters:**  
RBAC and least-privilege are foundational IAM requirements in every job description. Custom roles show you understand the *engineering* side, not just built-in role assignments.

**Resume/LinkedIn bullet:**
> *Engineered custom RBAC roles and Administrative Unit scoping in Microsoft Entra ID to enforce least-privilege access, eliminating over-permissioned admin accounts and enabling delegated management without Global Admin.*

---

## 🛠️ Technical Environment

- **Identity Platform:** Microsoft Entra ID (Free/P1/P2 trial tenant)
- **Scripting:** PowerShell 7.x, Microsoft Graph PowerShell SDK
- **APIs:** Microsoft Graph API v1.0 / beta
- **Monitoring:** Microsoft Sentinel, Entra ID Sign-in Logs, KQL
- **Standards:** NIST SP 800-53 Rev. 5, NIST SP 800-63B, HSPD-12, Zero Trust Architecture (NIST SP 800-207)
- **Certifications (in progress):** SC-300 Microsoft Identity and Access Administrator

---

## 📈 Skills Demonstrated Across Projects

| Skill | Projects |
|-------|---------|
| Microsoft Entra ID Administration | All |
| Conditional Access Policy Design | 1 |
| PowerShell + MS Graph Automation | 2, 8 |
| Privileged Identity Management (PIM) | 3 |
| Identity Governance & Administration (IGA) | 4 |
| SCIM / SAML / OIDC / SSO | 5 |
| KQL / Microsoft Sentinel | 6 |
| FIDO2 / Passwordless | 7 |
| RBAC / Custom Role Engineering | 8 |
| NIST 800-53 Control Mapping | 1, 3, 7 |
| Zero Trust Architecture | 1, 3 |

---

## 📬 Contact

**Jonathan Dada**  
IAM / ICAM Professional — Ernst & Young (DHS)  
📧 jdada2003@gmail.com  
🔗 [LinkedIn](https://www.linkedin.com/in/jonathan-dada123)  
📍 Severn, MD (DC Metro Area)

---

*Built to demonstrate enterprise IAM engineering skills aligned with SC-300, federal ICAM standards, and private sector IAM engineer role requirements.*
