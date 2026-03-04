# Active Directory Troubleshooting Labs

This repository contains hands-on troubleshooting scenarios performed in a Windows Active Directory lab.

The goal is to practice identifying problems across common infrastructure layers:

- Network connectivity
- DNS resolution
- Active Directory authentication
- Group membership and tokens
- File share permissions
- Account lockouts

Each lab intentionally breaks a service and walks through the troubleshooting process step-by-step.

This mimics real-world helpdesk and system administrator troubleshooting workflows.

---

## Lab List

| Lab | Scenario | Focus |
|----|----|----|
| 01 | No Internet | DNS vs Routing |
| 02 | Access Denied to Folder | AD Groups / NTFS |
| 03 | Cannot Login | Account Lockout |
| 04 | Cannot Print | Spooler / Print Server |
| 05 | Email Not Working | DNS / Authentication |

---

## Lab Environment

Domain: `corp.local`

Servers:

| Host | Role |
|-----|-----|
| DC01 | Domain Controller |
| DC02 | Domain Controller |
| FS01 | File Server |
| CL01 | Domain Client |

---

## Goal

Develop a repeatable troubleshooting methodology:

1. Confirm scope
2. Identify system layer
3. Run targeted checks
4. Fix root cause
5. Verify resolution
6. Document findings
