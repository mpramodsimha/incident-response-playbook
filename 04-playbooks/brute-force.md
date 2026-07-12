
# Brute Force Attack — IR Playbook

> A complete, practical guide for SOC analysts — from detection to escalation.
> Based on real-world SOC experience across MSSPs, startups, and Fortune 500 companies.

---

## What is a Brute Force Attack?

Attacker tries thousands of username/password combinations to break into an account. Simple, noisy, but still works because people use weak passwords.

**Common Types:**
- **Simple brute force** — try every possible combination
- **Dictionary attack** — use a wordlist (rockyou.txt, etc.)
- **Credential stuffing** — use leaked creds from breaches
- **Password spraying** — one password across many accounts (avoids lockout)

---

## MITRE ATT&CK Mapping

| Technique ID | Name | Description |
|---|---|---|
| T1110 | Brute Force | Parent technique |
| T1110.001 | Password Guessing | Systematic guess attempts |
| T1110.002 | Password Cracking | Crack hashed credentials offline |
| T1110.003 | Password Spraying | One password across many accounts |
| T1110.004 | Credential Stuffing | Use leaked breached credentials |

### Full ATT&CK Kill Chain

| Phase | Technique | What to Look For |
|---|---|---|
| Initial Access | T1078 — Valid Accounts | Successful login after brute force |
| Credential Access | T1110 — Brute Force | Multiple 4625 events |
| Lateral Movement | T1021 — Remote Services | RDP/SMB from compromised host |
| Persistence | T1053 — Scheduled Task | New tasks created post-compromise |
| Defense Evasion | T1070 — Log Clearing | Security log wiped after access |

---

## How You'll See It — Alert Sources

| Source | What You'll See |
|---|---|
| SIEM (Splunk/Sentinel) | Multiple 4625 events from same source IP |
| CrowdStrike | "Brute Force Attempt" or "Multiple Failed Logins" detection |
| Azure AD / Entra ID | Risky sign-in, unfamiliar location |
| Firewall | Repeated connections to port 3389 (RDP), 22 (SSH), 443 (OWA) |
| Proofpoint/Email Gateway | Multiple failed O365 auth attempts |

---

## Raw Log — What It Looks Like

### Windows Event ID 4625 (Failed Logon)

```xml
<Event>
  <System>
    <EventID>4625</EventID>
    <TimeCreated SystemTime="2026-07-11T03:14:22.123Z"/>
  </System>
  <EventData>
    <Data Name="TargetUserName">jsmith</Data>
    <Data Name="TargetDomainName">CORP</Data>
    <Data Name="Status">0xC000006D</Data>
    <Data Name="SubStatus">0xC000006A</Data>
    <Data Name="IpAddress">185.220.101.45</Data>
    <Data Name="IpPort">49832</Data>
    <Data Name="LogonType">10</Data>
    <Data Name="WorkstationName">UNKNOWN</Data>
  </EventData>
</Event>
