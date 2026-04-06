# Project 2: Automate User Lifecycle with Microsoft Entra ID

## Overview
Simulated a full joiner/mover/leaver (JML) identity lifecycle using Microsoft Entra ID's free tier. Created users, assigned them to applications, and then disabled and deleted them — replicating the exact workflow used in enterprise IAM to close orphaned accounts, one of the most common attack vectors in breach reports.

**Why it matters:** Orphaned accounts are consistently listed as a top attack vector. This lab shows you understand how to manage the full identity lifecycle, not just provisioning.

---

## Tools & Technologies
- Microsoft Entra ID (free tier)
- Microsoft Graph API
- PowerShell
- Entra ID Portal

---

## What I Built

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

---

## Key Concepts Practiced
- Joiner/Mover/Leaver (JML) lifecycle management
- Application assignment and role-based access
- Session revocation and account disabling
- Orphaned account prevention
- Access removal verification

---

## Screenshots
> *(Add screenshots of: user creation, app assignment, account disable confirmation, deletion confirmation)*
