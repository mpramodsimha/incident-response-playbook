Incident Response Playbook
Brute Force Attack
A complete, practical guide for SOC analysts — from detection to escalation
Based on real-world SOC experience across MSSPs, startups, and Fortune 500 companies


1. What is a Brute Force Attack?
Attacker tries thousands of username/password combinations to break into an account. Simple, noisy, but still works because people use weak passwords.
Common Types:
•	Simple brute force — try every possible combination
•	Dictionary attack — use a wordlist (rockyou.txt, etc.)
•	Credential stuffing — use leaked creds from breaches
•	Password spraying — one password across many accounts (avoids lockout)

2. MITRE ATT&CK Mapping
Technique ID	Name	Sub-Technique	Description
T1110	Brute Force	—	Parent technique
T1110.001	Password Guessing	Sub-technique	Systematic guess attempts
T1110.002	Password Cracking	Sub-technique	Crack hashed credentials offline
T1110.003	Password Spraying	Sub-technique	One password across many accounts
T1110.004	Credential Stuffing	Sub-technique	Use leaked breached credentials

Full ATT&CK Kill Chain
Phase	Technique	What to Look For
Initial Access	T1078 — Valid Accounts	Successful login after brute force
Credential Access	T1110 — Brute Force	Multiple 4625 events
Lateral Movement	T1021 — Remote Services	RDP/SMB from compromised host
Persistence	T1053 — Scheduled Task	New tasks created post-compromise
Defense Evasion	T1070 — Log Clearing	Security log wiped after access

3. How You'll See It — Alert Sources
Source	What You'll See
SIEM (Splunk/Sentinel)	Multiple 4625 events from same source IP
CrowdStrike	"Brute Force Attempt" or "Multiple Failed Logins" detection
Azure AD / Entra ID	Risky sign-in, unfamiliar location
Firewall	Repeated connections to port 3389 (RDP), 22 (SSH), 443 (OWA)
Proofpoint/Email Gateway	Multiple failed O365 auth attempts

4. Raw Log — What It Looks Like
Windows Event ID 4625 (Failed Logon)
Raw Log XML — Windows Event ID 4625
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

How to read this:
Field	Meaning
EventID 4625	Failed logon event
Status 0xC000006D	Bad username or password
SubStatus 0xC000006A	Wrong password (username exists)
LogonType 10	Remote Desktop (RDP)
IpAddress	Where the attempt came from
WorkstationName UNKNOWN	Not a domain-joined machine — suspicious

5. True Positive vs False Positive — How to Tell the Difference
Not every brute force alert is an attack. Employees forget passwords all the time.
Signs It's a FALSE POSITIVE (Legitimate User)
Indicator	Why
Source IP is internal/VPN	Employee on corporate network
3–5 failures then a success	Forgot password, tried a few times
Happens during business hours	Normal work pattern
User calls helpdesk around same time	They know they are locked out
LogonType 2 (Interactive/Console)	Sitting at their own machine
Account matches the workstation owner	It's their own device
SubStatus 0xC0000071 (password expired)	Password rotation issue

Signs It's a TRUE POSITIVE (Attack)
Indicator	Why
Source IP is external/unknown geo	Not the employee
100+ failures in minutes	No human types that fast
Happens at 2–4 AM	Nobody's working
Targets multiple accounts from same IP	Spraying
LogonType 10 (RDP) or 3 (Network) from outside	Remote attack
WorkstationName = UNKNOWN	Not a domain-joined machine
SubStatus 0xC000006A repeated 100x	Automated tool
Successful logon after failures from same IP	They got in
No helpdesk ticket from the user	User didn't do this

Quick Decision Tree
Quick Decision Tree — Is This Real?
Alert fires: Multiple 4625 events
│
├── Source IP internal + < 10 attempts + business hours?
│   └── YES → Likely forgot password. Check with user. Close as FP.
│
├── Source IP external + > 20 attempts + odd hours?
│   └── YES → Likely attack. Investigate immediately.
│
├── One IP hitting MANY accounts?
│   └── YES → Password spray. Treat as TP.
│
└── Successful 4624 after the failures from same IP?
    └── YES → Confirmed compromise. Escalate NOW.

Real-World Examples
Example 1 — False Positive
Monday 9:15 AM. User jsmith has 4 failed logins (4625) from IP 10.0.5.22 (their assigned workstation). Then a successful login (4624). SubStatus shows 0xC0000071 (expired password).
Verdict: Password expired over the weekend. They tried their old password a few times. Close as FP.

Example 2 — True Positive
Saturday 3:22 AM. 847 failed logins (4625) for user svc_backup from IP 185.220.101.45 (Tor exit node). LogonType 10 (RDP). Then one successful login (4624) at 3:30 AM.
Verdict: Brute force attack succeeded. Disable account, isolate endpoint, escalate.

Example 3 — Needs More Context
Tuesday 6 PM. User admin_jones has 15 failed logins from VPN IP. No success. User is remote worker in different timezone.
Verdict: Could be either. Contact the user to verify. Check if their VPN session was active. If user confirms it wasn't them → treat as TP.

6. Investigation Steps (What You Actually Do)
Step 1: Confirm It's Real
SPL Query — Confirm Brute Force
index=windows EventCode=4625
| stats count by src_ip, user, dest
| where count > 10
| sort -count
Ask yourself:
•	Is it one IP hitting one account? → Targeted brute force
•	Is it one IP hitting many accounts? → Password spraying
•	Is it many IPs hitting one account? → Distributed/botnet attack

Step 2: Check if They Got In
SPL Query — Check for Successful Logon (4624)
index=windows EventCode=4624 src_ip="185.220.101.45"
| table _time, user, src_ip, LogonType, dest
If you see a 4624 (successful logon) from the same IP after the failures — the attacker got in. Escalate immediately.

Step 3: Threat Intel on Source IP
•	Check in your TI platform (OpenCTI, VirusTotal, AbuseIPDB)
•	Is it a known Tor exit node? VPN? Hosting provider?
•	Geo-location — does it match where the user normally logs in?

Step 4: Check the Target Account
•	Is it a service account? Admin? Regular user?
•	Is MFA enabled on this account?
•	Has this user reported anything suspicious?

Step 5: Scope the Attack
SPL Query — Scope the Attack
index=windows EventCode=4625 src_ip="185.220.101.45"
| stats count dc(user) as unique_users by src_ip
| table src_ip, count, unique_users
How many accounts did they try? One account = targeted. Many accounts = spray.

7. Containment — What to Do
If They Did NOT Get In
1.	Block the source IP at firewall/WAF
2.	Verify account lockout policy is working
3.	Confirm MFA is enabled on targeted accounts
4.	Add IP to watchlist in SIEM
5.	Document and close as contained

If They DID Get In (4624 after 4625) 
1.	Immediately disable the compromised account
2.	Isolate the endpoint (CrowdStrike → Network Contain)
3.	Force password reset
4.	Revoke all active sessions/tokens
5.	Check for persistence:
◦	New scheduled tasks?
◦	New services installed?
◦	Registry run keys modified?
◦	Any lateral movement (4624 Type 3 from that host)?
6.	Escalate to Incident Commander

8. Detection Rules
Splunk — Basic Brute Force
SPL — Basic Brute Force Detection
index=windows EventCode=4625
| bucket _time span=5m
| stats count by src_ip, user, _time
| where count > 20

Splunk — Password Spray Detection
SPL — Password Spray Detection
index=windows EventCode=4625
| bucket _time span=10m
| stats dc(user) as unique_users count by src_ip, _time
| where unique_users > 5 AND count > 20

Splunk — Brute Force Followed by Success
SPL — Brute Force Followed by Successful Logon
index=windows (EventCode=4625 OR EventCode=4624)
| transaction src_ip maxspan=30m
| where eventcount > 10 AND EventCode=4624
| table _time, src_ip, user, dest

KQL (Microsoft Sentinel) — Brute Force
KQL — Microsoft Sentinel Brute Force Detection
SecurityEvent
| where EventID == 4625
| summarize FailedAttempts = count() by SourceIP = IpAddress,
    TargetAccount = TargetUserName, bin(TimeGenerated, 5m)
| where FailedAttempts > 20

Sigma Rule
Sigma Rule — Brute Force Multiple Failed Logons
title: Brute Force - Multiple Failed Logons
id: 1a2b3c4d-5e6f-7890-abcd-ef1234567890
status: experimental
logsource:
  product: windows
  service: security
detection:
  selection:
    EventID: 4625
  condition: selection | count(TargetUserName) by IpAddress > 20
  timeframe: 5m
level: medium
tags:
  - attack.credential_access
  - attack.t1110

9. Common Mistakes Analysts Make
❌ Mistake	Why It Matters
Closing the alert just because the account locked out	Attacker might try another account or already succeeded
Not checking if a successful login followed the failures	The whole point — 4624 after 4625 = compromise
Ignoring LogonType	Type 10 = RDP from outside is far worse than Type 2 = local console
Not checking if the source IP hit other accounts too	Could be a spray attack targeting the whole org
Forgetting to check for lateral movement after a successful compromise	Attacker may have already moved to other systems

10. Escalation Criteria
Escalate to P1 / Incident Commander if ANY of the following are true:
1.	Successful logon confirmed after brute force (4624 from attacker IP)
2.	Target is a privileged account (admin, service account, domain admin)
3.	Multiple accounts compromised
4.	Evidence of lateral movement post-compromise
5.	Source is internal IP (could mean attacker already inside the network)

11. Communication Template
To: SOC Manager / Incident Commander
Email Communication Template — Incident Notification
Subject: [P2] Brute Force Attack — Successful Compromise Detected

Summary: Multiple failed logon attempts (4625) detected from IP 185.220.101.45
targeting user jsmith@corp.com via RDP. A successful logon (4624) was observed
at 03:22 UTC following 847 failed attempts over 8 minutes.

Actions Taken:
  - Account disabled
  - Endpoint isolated via CrowdStrike
  - Source IP blocked at perimeter firewall
  - Password reset initiated

Next Steps:
  - Forensic review of endpoint for persistence
  - Check for lateral movement
  - Awaiting TI enrichment on source IP

12. Practice Challenge
📊 Scenario: You see this in Splunk at 2 AM
• 1,200 Event ID 4625 from IP 91.240.118.72 in 10 minutes
• All targeting different accounts in the CORP domain
• LogonType = 3 (Network)
• Then one Event ID 4624 for user svc_backup from same IP

Questions:
6.	What type of brute force is this?
7.	What's your first action?
8.	Why is svc_backup being compromised especially dangerous?
9.	What do you check next on the endpoint?
10.	What's your escalation path?

✅ Answers
1. Password spraying (many accounts, one IP)
2. Disable svc_backup account immediately + isolate endpoint
3. Service accounts often have elevated privileges and no MFA
4. Check for scheduled tasks, new services, lateral movement (4624 Type 3 from that host to others)
5. P1 escalation — privileged account compromised via spray attack

Incident Response Playbook — Brute Force Attack
Built for SOC analysts — from detection to escalation. Part of the incident-response-playbook repository.

