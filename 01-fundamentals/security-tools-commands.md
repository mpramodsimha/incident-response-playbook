
# Networking & Security Tools — Command Line Basics

> These are the first tools every SOC analyst and IT professional must know.
> You use these daily to troubleshoot network issues, investigate incidents, and verify connectivity.

---

## 1. PING

### What Does It Do?

Ping checks if a computer/server is **reachable** and **how fast** the connection is. It sends a small packet (like knocking on a door) and waits for a reply.

> **Real-life analogy:** You call your friend's phone. If they pick up, you know they're available. If it rings and rings with no answer, something is wrong.

### When Do SOC Analysts Use It?

- Check if a server is up or down during an incident
- Verify if a blocked IP is actually unreachable after firewall rule
- Test network connectivity after isolating a compromised host
- Quick health check on critical systems

### How to Use It

    ping cyberdx.net
    ping 192.168.1.1
    ping 10.0.5.22 -t        (continuous ping — keeps going until you press Ctrl+C)
    ping 10.0.5.22 -n 10     (send exactly 10 pings)

### Sample Output

    C:\Users\Analyst>ping cyberdx.net

    Pinging cyberdx.net [13.248.243.5] with 32 bytes of data:
    Reply from 13.248.243.5: bytes=32 time=22ms TTL=244
    Reply from 13.248.243.5: bytes=32 time=16ms TTL=244
    Reply from 13.248.243.5: bytes=32 time=38ms TTL=244
    Reply from 13.248.243.5: bytes=32 time=14ms TTL=244

    Ping statistics for 13.248.243.5:
        Packets: Sent = 4, Received = 4, Lost = 0 (0% loss),
    Approximate round trip times in milli-seconds:
        Minimum = 14ms, Maximum = 38ms, Average = 22ms

### How to Read the Output

| Field | What It Means |
|---|---|
| Reply from 13.248.243.5 | The server responded — it's alive and reachable |
| bytes=32 | Size of the ICMP packet sent |
| time=22ms | Round-trip time — lower is faster |
| TTL=244 | Time To Live — how many hops remain before packet dies |
| Packets Lost = 0 | All packets came back — connection is healthy |

### What Bad Output Looks Like

    C:\Users\Analyst>ping 10.0.5.99

    Pinging 10.0.5.99 with 32 bytes of data:
    Request timed out.
    Request timed out.
    Request timed out.
    Request timed out.

    Ping statistics for 10.0.5.99:
        Packets: Sent = 4, Received = 0, Lost = 4 (100% loss)

> **This means:** The target is either offline, blocked by a firewall, or doesn't exist.

### How Ping Works (Flow)

    Your Computer                          Target Server
         |                                       |
         |  1. Sends ICMP Echo Request           |
         |  ──────────────────────────────────>  |
         |                                       |
         |  2. Server receives the request       |
         |                                       |
         |  3. Sends ICMP Echo Reply             |
         |  <──────────────────────────────────  |
         |                                       |
         |  4. Your computer calculates          |
         |     the round-trip time (22ms)        |
         |                                       |
    
    If no reply comes back = host is down or blocked

### Common Ping Errors and What They Mean

| Error | Meaning |
|---|---|
| Request timed out | Target didn't respond — could be down or firewall blocking |
| Destination host unreachable | No route to that IP — network path doesn't exist |
| TTL expired in transit | Packet bounced around too many routers and died |
| General failure | Your own network adapter has a problem |

---

## 2. IPCONFIG

### What Does It Do?

Shows your computer's **network configuration** — your IP address, subnet mask, default gateway, and DNS servers. Think of it as your computer's "network ID card."

> **Real-life analogy:** Like checking your home address, zip code, and which road leads to the highway. You need to know YOUR address before you can troubleshoot why you can't reach someone else.

### When Do SOC Analysts Use It?

- Check if a machine has the correct IP address
- Verify DNS settings during a DNS poisoning investigation
- See if DHCP assigned an unexpected IP (rogue DHCP server?)
- Flush DNS cache after blocking a malicious domain
- Identify which network adapter is active

### How to Use It

| Command | Purpose |
|---|---|
| ipconfig | Basic IP configuration |
| ipconfig /all | Full details — DNS, DHCP, MAC address |
| ipconfig /flushdns | Clear DNS cache — useful after blocking domains |
| ipconfig /release | Give up your current IP |
| ipconfig /renew | Get a new IP from DHCP |

### Sample Output — ipconfig

    C:\Users\Analyst>ipconfig

    Windows IP Configuration

    Ethernet adapter Ethernet:

       Connection-specific DNS Suffix  . : cyberdx.net
       IPv4 Address. . . . . . . . . . . : 10.0.5.22
       Subnet Mask . . . . . . . . . . . : 255.255.255.0
       Default Gateway . . . . . . . . . : 10.0.5.1

    Wireless LAN adapter Wi-Fi:

       Connection-specific DNS Suffix  . : cyberdx.net
       IPv4 Address. . . . . . . . . . . : 192.168.1.105
       Subnet Mask . . . . . . . . . . . : 255.255.255.0
       Default Gateway . . . . . . . . . : 192.168.1.1

### Sample Output — ipconfig /all

    C:\Users\Analyst>ipconfig /all

    Ethernet adapter Ethernet:

       Host Name . . . . . . . . . . . . : ANALYST-PC
       Physical Address. . . . . . . . . : 00-1A-2B-3C-4D-5E    <-- MAC Address
       DHCP Enabled. . . . . . . . . . . : Yes
       IPv4 Address. . . . . . . . . . . : 10.0.5.22
       Subnet Mask . . . . . . . . . . . : 255.255.255.0
       Default Gateway . . . . . . . . . : 10.0.5.1
       DNS Servers . . . . . . . . . . . : 10.0.1.10
                                            10.0.1.11
       DHCP Server . . . . . . . . . . . : 10.0.1.5
       Lease Obtained. . . . . . . . . . : Friday, July 11, 2026 8:00:00 AM
       Lease Expires . . . . . . . . . . : Saturday, July 12, 2026 8:00:00 AM

### How to Read the Output

| Field | What It Means |
|---|---|
| Connection-specific DNS Suffix (cyberdx.net) | The DNS domain assigned to your network |
| IPv4 Address (10.0.5.22) | Your computer's address on the network |
| Subnet Mask (255.255.255.0) | Defines your network boundary — who's "local" to you |
| Default Gateway (10.0.5.1) | The router — all traffic to the outside goes through here |
| DNS Servers (10.0.1.10) | The server that translates names (google.com) to IPs |
| Physical Address (MAC) | Your network card's unique hardware ID |
| DHCP Enabled: Yes | Your IP was assigned automatically (not manually set) |

### How ipconfig Relates to Your Network (Flow)

    Your Computer (10.0.5.22)
         |
         | All traffic goes to...
         v
    Default Gateway / Router (10.0.5.1)
         |
         +------------+------------+
         |                         |
         | Routes DNS Requests     | Routes Internet Traffic
         v                         v
    DNS Server                  Internet
    (10.0.1.10)                 (Websites, APIs)
         |
         | Resolves domain names
         | cyberdx.net --> 10.0.5.50
         | google.com  --> 142.250.80.46
         v
    Response comes back the same path to your computer

### Security Use Cases

| Scenario | What to Check |
|---|---|
| Suspected DNS poisoning | ipconfig /all — are DNS servers correct? Or pointing to attacker's server? |
| Machine got weird IP | ipconfig — is it in the right subnet? Rogue DHCP? |
| Blocked a malicious domain but user can still reach it | ipconfig /flushdns — clear cached DNS entries |
| Investigating lateral movement | ipconfig on compromised host — what network is it on? |
| Suspected rogue DHCP server | ipconfig /all — verify DHCP Server matches approved infrastructure |
| Incident response host identification | Collect IP, subnet, gateway, DNS, DHCP info using ipconfig /all |

---

## 3. NSLOOKUP

### What Does It Do?

Looks up **DNS records** — translates a domain name (like google.com) to an IP address, or vice versa. DNS is like the phone book of the internet.

> **Real-life analogy:** You know your friend's name but not their phone number. You look them up in your contacts (DNS) to find their number (IP address).

### When Do SOC Analysts Use It?

- Check if a suspicious domain resolves to a known malicious IP
- Verify DNS is working correctly during an outage
- Investigate phishing — where does the fake domain actually point?
- Check if a domain was recently registered (new = suspicious)
- Verify internal DNS records are correct

### How to Use It

| Command | Purpose | SOC Use Case |
|---|---|---|
| nslookup cyberdx.net | Basic DNS lookup | Verify domain resolution |
| nslookup 13.248.243.5 | Reverse lookup (IP to name) | Identify ownership of suspicious IPs |
| nslookup -type=MX cyberdx.net | Query mail exchange records | Email security investigations |
| nslookup -type=TXT cyberdx.net | Query TXT records | Check SPF, DKIM records |
| nslookup -type=NS cyberdx.net | Query name server records | Identify authoritative DNS servers |

### Sample Output — Normal Lookup

    C:\Users\Analyst>nslookup cyberdx.net

    Server:  cdns01.comcast.net
    Address:  75.75.75.75

    Non-authoritative answer:
    Name:    cyberdx.net
    Addresses:  13.248.243.5
                76.223.105.230

### Sample Output — Suspicious Domain

    C:\Users\Analyst>nslookup totally-legit-microsoft-login.com

    Server:  dns-01.cyberdx.net
    Address:  10.0.1.10

    Non-authoritative answer:
    Name:    totally-legit-microsoft-login.com
    Address:  185.220.101.45

> **Red flag!** This "Microsoft login" page is hosted on IP 185.220.101.45 — which is a known Tor exit node. This is a phishing site!

### How to Read the Output

| Field | What It Means |
|---|---|
| Server: dns-01.cyberdx.net | Which DNS server answered your question |
| Address: 10.0.1.10 | The DNS server's IP |
| Non-authoritative answer | The answer came from cache, not directly from the domain's authoritative DNS |
| Name: cyberdx.net | The domain you asked about |
| Addresses: 13.248.243.5 | The IP address(es) that domain points to |

### How DNS Works (Flow)

    You type "google.com" in browser
         |
         | 1. Your computer asks: "What's the IP for google.com?"
         v
    Local DNS Cache (your computer)
         |
         | Not found in cache?
         v
    Company DNS Server (10.0.1.10)
         |
         | Not found there either?
         v
    Root DNS Server --> .com DNS Server --> Google's DNS Server
         |
         | 2. Answer comes back: "google.com = 142.250.80.46"
         v
    Your computer connects to 142.250.80.46
         |
         | 3. Google's website loads
         v
    You see the webpage

### Security Use Cases

| Scenario | Command | What You're Looking For |
|---|---|---|
| Phishing email has suspicious link | nslookup suspicious-domain.com | Does it resolve to a known bad IP? |
| User visited malware site | nslookup malware-site.com | What IP was serving the malware? |
| DNS poisoning suspected | nslookup cyberdx.net | Is it resolving to the WRONG IP? |
| Email spoofing investigation | nslookup -type=TXT cyberdx.net | Check SPF records |

---

## 4. TRACERT (Traceroute)

### What Does It Do?

Shows the **exact path** your network traffic takes to reach a destination — every router (hop) along the way. Like GPS showing every turn on your route.

> **Real-life analogy:** If ping is "Can I reach my friend's house?" then tracert is "Show me every street and intersection I pass through to get there."

### When Do SOC Analysts Use It?

- Find WHERE in the network path a connection is failing
- Identify if traffic is being routed through unexpected countries (man-in-the-middle?)
- Troubleshoot why a server is slow (which hop has high latency?)
- Verify traffic is going through the correct firewall/proxy
- Investigate if traffic is being redirected (BGP hijacking)

### How to Use It

| Command | Purpose |
|---|---|
| tracert google.com | Trace route to google |
| tracert 10.0.5.50 | Trace route to internal server |
| tracert -d google.com | Skip DNS resolution — faster |

### Sample Output

    C:\Users\Analyst>tracert google.com

    Tracing route to google.com [142.250.80.46]
    over a maximum of 30 hops:

      1     1 ms     1 ms     1 ms   10.0.5.1          (your gateway/router)
      2     3 ms     2 ms     3 ms   10.0.1.1          (core switch)
      3    10 ms     9 ms    10 ms   203.0.113.1       (ISP router)
      4    12 ms    11 ms    12 ms   72.14.215.85      (Google's network)
      5    11 ms    12 ms    11 ms   142.250.80.46     (destination!)

    Trace complete.

### How to Read the Output

| Column | What It Means |
|---|---|
| Hop number (1, 2, 3...) | Each router/device the packet passes through |
| Time (1 ms, 3 ms, 10 ms) | How long it took to reach that hop (3 measurements) |
| IP Address | The router's IP at that hop |
| * * * Request timed out | That router doesn't respond to tracert (firewall blocking — not necessarily bad) |

### What Bad Output Looks Like

    C:\Users\Analyst>tracert internal-server.cyberdx.net

      1     1 ms     1 ms     1 ms   10.0.5.1
      2     3 ms     2 ms     3 ms   10.0.1.1
      3     *        *        *      Request timed out.
      4     *        *        *      Request timed out.
      5     *        *        *      Request timed out.

> **This means:** Traffic is being blocked after hop 2. A firewall or ACL is stopping the connection.

### How Tracert Works (Flow)

    Your Computer
         |
         | Sends packet with TTL=1 (dies at first router)
         v
    Hop 1: Router 10.0.5.1 --> sends back "I'm here!" (1ms)
         |
         | Sends packet with TTL=2 (dies at second router)
         v
    Hop 2: Router 10.0.1.1 --> sends back "I'm here!" (3ms)
         |
         | Sends packet with TTL=3 (dies at third router)
         v
    Hop 3: ISP Router 203.0.113.1 --> sends back "I'm here!" (10ms)
         |
         | Sends packet with TTL=4
         v
    Hop 4: Google Router --> sends back "I'm here!" (12ms)
         |
         | Sends packet with TTL=5
         v
    Hop 5: Destination 142.250.80.46 --> "You've arrived!" (11ms)

    TTL = Time To Live. Each router decreases it by 1.
    When TTL hits 0, that router sends back an error message.
    That's how tracert discovers each hop!

### Security Use Cases

| Scenario | What to Look For |
|---|---|
| Traffic going through unexpected country | Hops showing IPs from countries you don't expect |
| Connection to server is slow | Which hop has a big jump in latency? That's the bottleneck |
| Firewall rule verification | After blocking, tracert should stop at the firewall hop |
| Man-in-the-middle suspected | Extra unexpected hops that shouldn't be there |

---

## 5. NETSTAT

### What Does It Do?

Shows all **active network connections** on your computer — what's connected, what's listening, and which programs are talking to the internet.

> **Real-life analogy:** Like checking your phone's screen time — it shows you every app that's currently using the internet and who it's talking to.

### When Do SOC Analysts Use It?

- Check if malware is connecting to a Command & Control (C2) server
- See what ports are open on a compromised machine
- Identify suspicious outbound connections
- Verify if a service is running and listening
- Find which process is making a suspicious connection

### How to Use It

    netstat                    (show active connections)
    netstat -an               (show all connections with IP addresses, no DNS names)
    netstat -ano              (show all connections + which process ID owns them)
    netstat -b                (show which PROGRAM is making each connection — run as Admin)
    netstat -an | find "ESTABLISHED"    (show only active connections)
    netstat -an | find "4444"           (look for a specific suspicious port)

### Sample Output — netstat -ano

    C:\Users\Analyst>netstat -ano

    Active Connections

      Proto  Local Address          Foreign Address        State           PID
      TCP    0.0.0.0:80             0.0.0.0:0              LISTENING       4
      TCP    0.0.0.0:443            0.0.0.0:0              LISTENING       4
      TCP    10.0.5.22:49832        142.250.80.46:443      ESTABLISHED     8812
      TCP    10.0.5.22:49900        52.96.166.130:443      ESTABLISHED     12456
      TCP    10.0.5.22:50100        185.220.101.45:4444    ESTABLISHED     6660
      TCP    0.0.0.0:3389           0.0.0.0:0              LISTENING       1200

### How to Read the Output

| Column | What It Means |
|---|---|
| Proto | Protocol — TCP or UDP |
| Local Address | Your computer's IP and port |
| Foreign Address | The remote server's IP and port you're connected to |
| State | Connection status (see below) |
| PID | Process ID — which program owns this connection |

### Connection States Explained

| State | What It Means |
|---|---|
| LISTENING | Port is open and waiting for connections (like a door that's unlocked) |
| ESTABLISHED | Active connection — data is flowing right now |
| TIME_WAIT | Connection just closed, waiting to fully terminate |
| CLOSE_WAIT | Remote side closed, your side hasn't yet |
| SYN_SENT | Trying to connect (handshake started) |

### Spotting Suspicious Connections

Look at the sample output above. Can you spot the suspicious one?

    TCP    10.0.5.22:50100        185.220.101.45:4444    ESTABLISHED     6660

**Why this is suspicious:**

- **185.220.101.45** — This is a known Tor exit node (attacker IP)
- **Port 4444** — This is the default port for Metasploit reverse shell (hacking tool)
- **ESTABLISHED** — The connection is ACTIVE right now
- **PID 6660** — We need to find out what program this is

### How to Find What Program Is Making the Connection

    tasklist | find "6660"

Output:

    evil_malware.exe          6660 Console    1     15,232 K

> **Now you know:** Process ID 6660 is "evil_malware.exe" connecting to an attacker's server on port 4444. Kill it and investigate!

### How Netstat Helps in Incident Response (Flow)

    Alert: "Suspicious outbound connection detected"
         |
         | Step 1: Run netstat -ano on the machine
         v
    Find suspicious connection (weird IP, weird port)
         |
         | Step 2: Note the PID (Process ID)
         v
    Run: tasklist | find "PID"
         |
         | Step 3: Identify the program
         v
    Is it legitimate? (chrome.exe, outlook.exe = probably fine)
    Is it suspicious? (svchost.exe connecting to port 4444 = BAD)
         |
         | Step 4: If malicious
         v
    Kill the process --> Isolate the machine --> Investigate further

### Common Suspicious Ports to Watch For

| Port | What It Usually Means |
|---|---|
| 4444 | Metasploit default (hacking tool) |
| 5555 | Android Debug Bridge (if not expected) |
| 1337 | "Leet" — often used by hackers for fun |
| 8080 | Web proxy — could be C2 communication |
| 6667 | IRC — old school C2 channel |
| 443 to unknown IPs | Could be encrypted C2 hiding in HTTPS traffic |
| Any port > 49000 to external | Unusual — investigate what program is doing this |

---

## Quick Comparison — When to Use Which Tool

| I Want To... | Use This Command |
|---|---|
| Check if a server is alive | ping |
| See my own IP address and network info | ipconfig |
| Find what IP a domain points to | nslookup |
| See the path my traffic takes to reach a server | tracert |
| See what's currently connected to my computer | netstat |

---

## All Commands Cheat Sheet

    # Am I connected to the network?
    ipconfig

    # Is the server alive?
    ping 10.0.5.50

    # What IP does this domain resolve to?
    nslookup suspicious-domain.com

    # What path does my traffic take?
    tracert google.com

    # What connections are active on this machine?
    netstat -ano

    # What program is using a specific connection?
    tasklist | find "PID_NUMBER"

    # Clear DNS cache after blocking a domain
    ipconfig /flushdns

    # Continuous ping (monitor if server stays up)
    ping 10.0.5.50 -t

---

## How These Tools Work Together in an Investigation

> **Scenario:** User reports their computer is slow and acting weird.

    Step 1: ipconfig
       --> Check: Does the machine have the right IP? Right DNS server?
       --> Result: DNS is pointing to 185.220.101.45 (NOT our DNS server!)
       --> Conclusion: DNS might be hijacked!

    Step 2: nslookup portal.cyberdx.net
       --> Check: Is our company portal resolving to the right IP?
       --> Result: It's resolving to 91.240.118.72 (wrong IP!)
       --> Conclusion: DNS IS hijacked — user is being sent to fake sites

    Step 3: netstat -ano
       --> Check: What connections are active?
       --> Result: PID 6660 is connected to 185.220.101.45:4444
       --> Conclusion: Malware is running and talking to attacker

    Step 4: ping 10.0.1.10 (real DNS server)
       --> Check: Can we reach our real DNS?
       --> Result: Request timed out
       --> Conclusion: Something is blocking access to real DNS

    Step 5: tracert 10.0.1.10
       --> Check: Where is the traffic being blocked?
       --> Result: Traffic stops at hop 2 (10.0.5.1 — the gateway)
       --> Conclusion: Gateway/router might be compromised too

    FINAL ACTION: Isolate the machine, escalate to P1, investigate the gateway

---

## Practice Challenges

### Challenge 1

You run netstat -ano and see:

    TCP    10.0.5.22:51000    91.240.118.72:443    ESTABLISHED    3344

Is this suspicious? How would you investigate?

**Answer:** Potentially suspicious. Check what PID 3344 is (tasklist | find "3344"). Look up 91.240.118.72 on VirusTotal. If it's a known C2 server and the process is unusual — it's malware.

---

### Challenge 2

You run nslookup microsoft.com and get:

    Name:    microsoft.com
    Address:  185.220.101.45

Is this normal?

**Answer:** NO! Microsoft.com should NOT resolve to 185.220.101.45 (a Tor exit node). This means DNS is poisoned or hijacked. The user is being redirected to a fake Microsoft site. Investigate immediately.

---

### Challenge 3

You run tracert internal-server.cyberdx.net and see:

      1     1 ms    10.0.5.1
      2     *       Request timed out.
      3     *       Request timed out.

What does this tell you?

**Answer:** Traffic is being blocked after your gateway (10.0.5.1). Either a firewall rule is blocking it, the server is down, or there's a network issue between your subnet and the server's subnet. Check firewall rules and server status.

---

> **Part of the [incident-response-playbook](https://github.com/mpramodsimha/incident-response-playbook) repository.**
> Built for SOC analysts — from detection to escalation.



What Bad Output Looks Like

C:\Users\Analyst>ping 10.0.5.99

Pinging 10.0.5.99 with 32 bytes of data:
Request timed out.
Request timed out.
Request timed out.
Request timed out.

Ping statistics for 10.0.5.99:
    Packets: Sent = 4, Received = 0, Lost = 4 (100% loss)

This means: The target is either offline, blocked by a firewall, or doesn't exist.

This means: The target is either offline, blocked by a firewall, or doesn't exist.

How Ping Works (Flow)

Your Computer                          Target Server
     |                                       |
     |  1. Sends ICMP Echo Request           |
     |  ──────────────────────────────────>  |
     |                                       |
     |  2. Server receives the request       |
     |                                       |
     |  3. Sends ICMP Echo Reply             |
     |  <──────────────────────────────────  |
     |                                       |
     |  4. Your computer calculates          |
     |     the round-trip time (22ms)        |
     |                                       |

If no reply comes back = host is down or blocked

Common Ping Errors and What They Mean

ErrorMeaningRequest timed outTarget didn't respond — could be down or firewall blockingDestination host unreachableNo route to that IP — network path doesn't existTTL expired in transitPacket bounced around too many routers and diedGeneral failureYour own network adapter has a problem

ErrorMeaning

Error

Meaning

Request timed outTarget didn't respond — could be down or firewall blocking

Request timed out

Target didn't respond — could be down or firewall blocking

Destination host unreachableNo route to that IP — network path doesn't exist

Destination host unreachable

No route to that IP — network path doesn't exist

TTL expired in transitPacket bounced around too many routers and died

TTL expired in transit

Packet bounced around too many routers and died

General failureYour own network adapter has a problem

General failure

Your own network adapter has a problem

2. IPCONFIG

What Does It Do?

Shows your computer's network configuration — your IP address, subnet mask, default gateway, and DNS servers. Think of it as your computer's "network ID card."

Real-life analogy: Like checking your home address, zip code, and which road leads to the highway. You need to know YOUR address before you can troubleshoot why you can't reach someone else.

Real-life analogy: Like checking your home address, zip code, and which road leads to the highway. You need to know YOUR address before you can troubleshoot why you can't reach someone else.

When Do SOC Analysts Use It?

- Check if a machine has the correct IP address

- Verify DNS settings during a DNS poisoning investigation

- See if DHCP assigned an unexpected IP (rogue DHCP server?)

- Flush DNS cache after blocking a malicious domain

- Identify which network adapter is active

How to Use It

CommandPurposeipconfigBasic IP configurationipconfig /allFull details — DNS, DHCP, MAC addressipconfig /flushdnsClear DNS cache — useful after blocking domainsipconfig /releaseGive up your current IPipconfig /renewGet a new IP from DHCP

CommandPurpose

Command

Purpose

ipconfigBasic IP configuration

ipconfig

Basic IP configuration

ipconfig /allFull details — DNS, DHCP, MAC address

ipconfig /all

Full details — DNS, DHCP, MAC address

ipconfig /flushdnsClear DNS cache — useful after blocking domains

ipconfig /flushdns

Clear DNS cache — useful after blocking domains

ipconfig /releaseGive up your current IP

ipconfig /release

Give up your current IP

ipconfig /renewGet a new IP from DHCP

ipconfig /renew

Get a new IP from DHCP

Sample Output — ipconfig

C:\Users\Analyst>ipconfig

Windows IP Configuration

Ethernet adapter Ethernet:

   Connection-specific DNS Suffix  . : cyberdx.net
   IPv4 Address. . . . . . . . . . . : 10.0.5.22
   Subnet Mask . . . . . . . . . . . : 255.255.255.0
   Default Gateway . . . . . . . . . : 10.0.5.1

Wireless LAN adapter Wi-Fi:

   Connection-specific DNS Suffix  . : cyberdx.net
   IPv4 Address. . . . . . . . . . . : 192.168.1.105
   Subnet Mask . . . . . . . . . . . : 255.255.255.0
   Default Gateway . . . . . . . . . : 192.168.1.1

Sample Output — ipconfig /all

C:\Users\Analyst>ipconfig /all

Ethernet adapter Ethernet:

   Host Name . . . . . . . . . . . . : ANALYST-PC
   Physical Address. . . . . . . . . : 00-1A-2B-3C-4D-5E    <-- MAC Address
   DHCP Enabled. . . . . . . . . . . : Yes
   IPv4 Address. . . . . . . . . . . : 10.0.5.22
   Subnet Mask . . . . . . . . . . . : 255.255.255.0
   Default Gateway . . . . . . . . . : 10.0.5.1
   DNS Servers . . . . . . . . . . . : 10.0.1.10
                                        10.0.1.11
   DHCP Server . . . . . . . . . . . : 10.0.1.5
   Lease Obtained. . . . . . . . . . : Friday, July 11, 2026 8:00:00 AM
   Lease Expires . . . . . . . . . . : Saturday, July 12, 2026 8:00:00 AM

How to Read the Output

FieldWhat It MeansConnection-specific DNS Suffix (cyberdx.net)The DNS domain assigned to your networkIPv4 Address (10.0.5.22)Your computer's address on the networkSubnet Mask (255.255.255.0)Defines your network boundary — who's "local" to youDefault Gateway (10.0.5.1)The router — all traffic to the outside goes through hereDNS Servers (10.0.1.10)The server that translates names (google.com) to IPsPhysical Address (MAC)Your network card's unique hardware IDDHCP Enabled: YesYour IP was assigned automatically (not manually set)

FieldWhat It Means

Field

What It Means

Connection-specific DNS Suffix (cyberdx.net)The DNS domain assigned to your network

Connection-specific DNS Suffix (cyberdx.net)

The DNS domain assigned to your network

IPv4 Address (10.0.5.22)Your computer's address on the network

IPv4 Address (10.0.5.22)

Your computer's address on the network

Subnet Mask (255.255.255.0)Defines your network boundary — who's "local" to you

Subnet Mask (255.255.255.0)

Defines your network boundary — who's "local" to you

Default Gateway (10.0.5.1)The router — all traffic to the outside goes through here

Default Gateway (10.0.5.1)

The router — all traffic to the outside goes through here

DNS Servers (10.0.1.10)The server that translates names (google.com) to IPs

DNS Servers (10.0.1.10)

The server that translates names (google.com) to IPs

Physical Address (MAC)Your network card's unique hardware ID

Physical Address (MAC)

Your network card's unique hardware ID

DHCP Enabled: YesYour IP was assigned automatically (not manually set)

DHCP Enabled: Yes

Your IP was assigned automatically (not manually set)

How ipconfig Relates to Your Network (Flow)

Your Computer (10.0.5.22)
     |
     | All traffic goes to...
     v
Default Gateway / Router (10.0.5.1)
     |
     +------------+------------+
     |                         |
     | Routes DNS Requests     | Routes Internet Traffic
     v                         v
DNS Server                  Internet
(10.0.1.10)                 (Websites, APIs)
     |
     | Resolves domain names
     | cyberdx.net --> 10.0.5.50
     | google.com  --> 142.250.80.46
     v
Response comes back the same path to your computer

Security Use Cases

ScenarioWhat to CheckSuspected DNS poisoningipconfig /all — are DNS servers correct? Or pointing to attacker's server?Machine got weird IPipconfig — is it in the right subnet? Rogue DHCP?Blocked a malicious domain but user can still reach itipconfig /flushdns — clear cached DNS entriesInvestigating lateral movementipconfig on compromised host — what network is it on?Suspected rogue DHCP serveripconfig /all — verify DHCP Server matches approved infrastructureIncident response host identificationCollect IP, subnet, gateway, DNS, DHCP info using ipconfig /all

ScenarioWhat to Check

Scenario

What to Check

Suspected DNS poisoningipconfig /all — are DNS servers correct? Or pointing to attacker's server?

Suspected DNS poisoning

ipconfig /all — are DNS servers correct? Or pointing to attacker's server?

Machine got weird IPipconfig — is it in the right subnet? Rogue DHCP?

Machine got weird IP

ipconfig — is it in the right subnet? Rogue DHCP?

Blocked a malicious domain but user can still reach itipconfig /flushdns — clear cached DNS entries

Blocked a malicious domain but user can still reach it

ipconfig /flushdns — clear cached DNS entries

Investigating lateral movementipconfig on compromised host — what network is it on?

Investigating lateral movement

ipconfig on compromised host — what network is it on?

Suspected rogue DHCP serveripconfig /all — verify DHCP Server matches approved infrastructure

Suspected rogue DHCP server

ipconfig /all — verify DHCP Server matches approved infrastructure

Incident response host identificationCollect IP, subnet, gateway, DNS, DHCP info using ipconfig /all

Incident response host identification

Collect IP, subnet, gateway, DNS, DHCP info using ipconfig /all

3. NSLOOKUP

What Does It Do?

Looks up DNS records — translates a domain name (like google.com) to an IP address, or vice versa. DNS is like the phone book of the internet.

Real-life analogy: You know your friend's name but not their phone number. You look them up in your contacts (DNS) to find their number (IP address).

Real-life analogy: You know your friend's name but not their phone number. You look them up in your contacts (DNS) to find their number (IP address).

When Do SOC Analysts Use It?

- Check if a suspicious domain resolves to a known malicious IP

- Verify DNS is working correctly during an outage

- Investigate phishing — where does the fake domain actually point?

- Check if a domain was recently registered (new = suspicious)

- Verify internal DNS records are correct

How to Use It

CommandPurposeSOC Use Casenslookup cyberdx.netBasic DNS lookupVerify domain resolutionnslookup 13.248.243.5Reverse lookup (IP to name)Identify ownership of suspicious IPsnslookup -type=MX cyberdx.netQuery mail exchange recordsEmail security investigationsnslookup -type=TXT cyberdx.netQuery TXT recordsCheck SPF, DKIM recordsnslookup -type=NS cyberdx.netQuery name server recordsIdentify authoritative DNS servers

CommandPurposeSOC Use Case

Command

Purpose

SOC Use Case

nslookup cyberdx.netBasic DNS lookupVerify domain resolution

nslookup cyberdx.net

Basic DNS lookup

Verify domain resolution

nslookup 13.248.243.5Reverse lookup (IP to name)Identify ownership of suspicious IPs

nslookup 13.248.243.5

Reverse lookup (IP to name)

Identify ownership of suspicious IPs

nslookup -type=MX cyberdx.netQuery mail exchange recordsEmail security investigations

nslookup -type=MX cyberdx.net

Query mail exchange records

Email security investigations

nslookup -type=TXT cyberdx.netQuery TXT recordsCheck SPF, DKIM records

nslookup -type=TXT cyberdx.net

Query TXT records

Check SPF, DKIM records

nslookup -type=NS cyberdx.netQuery name server recordsIdentify authoritative DNS servers

nslookup -type=NS cyberdx.net

Query name server records

Identify authoritative DNS servers

Sample Output — Normal Lookup

C:\Users\Analyst>nslookup cyberdx.net

Server:  cdns01.comcast.net
Address:  75.75.75.75

Non-authoritative answer:
Name:    cyberdx.net
Addresses:  13.248.243.5
            76.223.105.230

Sample Output — Suspicious Domain

C:\Users\Analyst>nslookup totally-legit-microsoft-login.com

Server:  dns-01.cyberdx.net
Address:  10.0.1.10

Non-authoritative answer:
Name:    totally-legit-microsoft-login.com
Address:  185.220.101.45

Red flag! This "Microsoft login" page is hosted on IP 185.220.101.45 — which is a known Tor exit node. This is a phishing site!

Red flag! This "Microsoft login" page is hosted on IP 185.220.101.45 — which is a known Tor exit node. This is a phishing site!

How to Read the Output

FieldWhat It MeansServer: dns-01.cyberdx.netWhich DNS server answered your questionAddress: 10.0.1.10The DNS server's IPNon-authoritative answerThe answer came from cache, not directly from the domain's authoritative DNSName: cyberdx.netThe domain you asked aboutAddresses: 13.248.243.5The IP address(es) that domain points to

FieldWhat It Means

Field

What It Means

Server: dns-01.cyberdx.netWhich DNS server answered your question

Server: dns-01.cyberdx.net

Which DNS server answered your question

Address: 10.0.1.10The DNS server's IP

Address: 10.0.1.10

The DNS server's IP

Non-authoritative answerThe answer came from cache, not directly from the domain's authoritative DNS

Non-authoritative answer

The answer came from cache, not directly from the domain's authoritative DNS

Name: cyberdx.netThe domain you asked about

Name: cyberdx.net

The domain you asked about

Addresses: 13.248.243.5The IP address(es) that domain points to

Addresses: 13.248.243.5

The IP address(es) that domain points to

How DNS Works (Flow)

You type "google.com" in browser
     |
     | 1. Your computer asks: "What's the IP for google.com?"
     v
Local DNS Cache (your computer)
     |
     | Not found in cache?
     v
Company DNS Server (10.0.1.10)
     |
     | Not found there either?
     v
Root DNS Server --> .com DNS Server --> Google's DNS Server
     |
     | 2. Answer comes back: "google.com = 142.250.80.46"
     v
Your computer connects to 142.250.80.46
     |
     | 3. Google's website loads
     v
You see the webpage

Security Use Cases

ScenarioCommandWhat You're Looking ForPhishing email has suspicious linknslookup suspicious-domain.comDoes it resolve to a known bad IP?User visited malware sitenslookup malware-site.comWhat IP was serving the malware?DNS poisoning suspectednslookup cyberdx.netIs it resolving to the WRONG IP?Email spoofing investigationnslookup -type=TXT cyberdx.netCheck SPF records

ScenarioCommandWhat You're Looking For

Scenario

Command

What You're Looking For

Phishing email has suspicious linknslookup suspicious-domain.comDoes it resolve to a known bad IP?

Phishing email has suspicious link

nslookup suspicious-domain.com

Does it resolve to a known bad IP?

User visited malware sitenslookup malware-site.comWhat IP was serving the malware?

User visited malware site

nslookup malware-site.com

What IP was serving the malware?

DNS poisoning suspectednslookup cyberdx.netIs it resolving to the WRONG IP?

DNS poisoning suspected

nslookup cyberdx.net

Is it resolving to the WRONG IP?

Email spoofing investigationnslookup -type=TXT cyberdx.netCheck SPF records

Email spoofing investigation

nslookup -type=TXT cyberdx.net

Check SPF records

4. TRACERT (Traceroute)

What Does It Do?

Shows the exact path your network traffic takes to reach a destination — every router (hop) along the way. Like GPS showing every turn on your route.

Real-life analogy: If ping is "Can I reach my friend's house?" then tracert is "Show me every street and intersection I pass through to get there."

Real-life analogy: If ping is "Can I reach my friend's house?" then tracert is "Show me every street and intersection I pass through to get there."

When Do SOC Analysts Use It?

- Find WHERE in the network path a connection is failing

- Identify if traffic is being routed through unexpected countries (man-in-the-middle?)

- Troubleshoot why a server is slow (which hop has high latency?)

- Verify traffic is going through the correct firewall/proxy

- Investigate if traffic is being redirected (BGP hijacking)

How to Use It

CommandPurposetracert google.comTrace route to googletracert 10.0.5.50Trace route to internal servertracert -d google.comSkip DNS resolution — faster

CommandPurpose

Command

Purpose

tracert google.comTrace route to google

tracert google.com

Trace route to google

tracert 10.0.5.50Trace route to internal server

tracert 10.0.5.50

Trace route to internal server

tracert -d google.comSkip DNS resolution — faster

tracert -d google.com

Skip DNS resolution — faster

Sample Output

C:\Users\Analyst>tracert google.com

Tracing route to google.com [142.250.80.46]
over a maximum of 30 hops:

  1     1 ms     1 ms     1 ms   10.0.5.1          (your gateway/router)
  2     3 ms     2 ms     3 ms   10.0.1.1          (core switch)
  3    10 ms     9 ms    10 ms   203.0.113.1       (ISP router)
  4    12 ms    11 ms    12 ms   72.14.215.85      (Google's network)
  5    11 ms    12 ms    11 ms   142.250.80.46     (destination!)

Trace complete.

How to Read the Output

ColumnWhat It MeansHop number (1, 2, 3...)Each router/device the packet passes throughTime (1 ms, 3 ms, 10 ms)How long it took to reach that hop (3 measurements)IP AddressThe router's IP at that hop* * * Request timed outThat router doesn't respond to tracert (firewall blocking — not necessarily bad)

ColumnWhat It Means

Column

What It Means

Hop number (1, 2, 3...)Each router/device the packet passes through

Hop number (1, 2, 3...)

Each router/device the packet passes through

Time (1 ms, 3 ms, 10 ms)How long it took to reach that hop (3 measurements)

Time (1 ms, 3 ms, 10 ms)

How long it took to reach that hop (3 measurements)

IP AddressThe router's IP at that hop

IP Address

The router's IP at that hop

* * * Request timed outThat router doesn't respond to tracert (firewall blocking — not necessarily bad)

* * * Request timed out

That router doesn't respond to tracert (firewall blocking — not necessarily bad)

What Bad Output Looks Like

C:\Users\Analyst>tracert internal-server.cyberdx.net

  1     1 ms     1 ms     1 ms   10.0.5.1
  2     3 ms     2 ms     3 ms   10.0.1.1
  3     *        *        *      Request timed out.
  4     *        *        *      Request timed out.
  5     *        *        *      Request timed out.

This means: Traffic is being blocked after hop 2. A firewall or ACL is stopping the connection.

This means: Traffic is being blocked after hop 2. A firewall or ACL is stopping the connection.

How Tracert Works (Flow)

Your Computer
     |
     | Sends packet with TTL=1 (dies at first router)
     v
Hop 1: Router 10.0.5.1 --> sends back "I'm here!" (1ms)
     |
     | Sends packet with TTL=2 (dies at second router)
     v
Hop 2: Router 10.0.1.1 --> sends back "I'm here!" (3ms)
     |
     | Sends packet with TTL=3 (dies at third router)
     v
Hop 3: ISP Router 203.0.113.1 --> sends back "I'm here!" (10ms)
     |
     | Sends packet with TTL=4
     v
Hop 4: Google Router --> sends back "I'm here!" (12ms)
     |
     | Sends packet with TTL=5
     v
Hop 5: Destination 142.250.80.46 --> "You've arrived!" (11ms)

TTL = Time To Live. Each router decreases it by 1.
When TTL hits 0, that router sends back an error message.
That's how tracert discovers each hop!

Security Use Cases

ScenarioWhat to Look ForTraffic going through unexpected countryHops showing IPs from countries you don't expectConnection to server is slowWhich hop has a big jump in latency? That's the bottleneckFirewall rule verificationAfter blocking, tracert should stop at the firewall hopMan-in-the-middle suspectedExtra unexpected hops that shouldn't be there

ScenarioWhat to Look For

Scenario

What to Look For

Traffic going through unexpected countryHops showing IPs from countries you don't expect

Traffic going through unexpected country

Hops showing IPs from countries you don't expect

Connection to server is slowWhich hop has a big jump in latency? That's the bottleneck

Connection to server is slow

Which hop has a big jump in latency? That's the bottleneck

Firewall rule verificationAfter blocking, tracert should stop at the firewall hop

Firewall rule verification

After blocking, tracert should stop at the firewall hop

Man-in-the-middle suspectedExtra unexpected hops that shouldn't be there

Man-in-the-middle suspected

Extra unexpected hops that shouldn't be there

5. NETSTAT

What Does It Do?

Shows all active network connections on your computer — what's connected, what's listening, and which programs are talking to the internet.

Real-life analogy: Like checking your phone's screen time — it shows you every app that's currently using the internet and who it's talking to.

Real-life analogy: Like checking your phone's screen time — it shows you every app that's currently using the internet and who it's talking to.

When Do SOC Analysts Use It?

- Check if malware is connecting to a Command & Control (C2) server

- See what ports are open on a compromised machine

- Identify suspicious outbound connections

- Verify if a service is running and listening

- Find which process is making a suspicious connection

How to Use It

netstat                    (show active connections)
netstat -an               (show all connections with IP addresses, no DNS names)
netstat -ano              (show all connections + which process ID owns them)
netstat -b                (show which PROGRAM is making each connection — run as Admin)
netstat -an | find "ESTABLISHED"    (show only active connections)
netstat -an | find "4444"           (look for a specific suspicious port)

Sample Output — netstat -ano

C:\Users\Analyst>netstat -ano

Active Connections

  Proto  Local Address          Foreign Address        State           PID
  TCP    0.0.0.0:80             0.0.0.0:0              LISTENING       4
  TCP    0.0.0.0:443            0.0.0.0:0              LISTENING       4
  TCP    10.0.5.22:49832        142.250.80.46:443      ESTABLISHED     8812
  TCP    10.0.5.22:49900        52.96.166.130:443      ESTABLISHED     12456
  TCP    10.0.5.22:50100        185.220.101.45:4444    ESTABLISHED     6660
  TCP    0.0.0.0:3389           0.0.0.0:0              LISTENING       1200

How to Read the Output

ColumnWhat It MeansProtoProtocol — TCP or UDPLocal AddressYour computer's IP and portForeign AddressThe remote server's IP and port you're connected toStateConnection status (see below)PIDProcess ID — which program owns this connection

ColumnWhat It Means

Column

What It Means

ProtoProtocol — TCP or UDP

Proto

Protocol — TCP or UDP

Local AddressYour computer's IP and port

Local Address

Your computer's IP and port

Foreign AddressThe remote server's IP and port you're connected to

Foreign Address

The remote server's IP and port you're connected to

StateConnection status (see below)

State

Connection status (see below)

PIDProcess ID — which program owns this connection

PID

Process ID — which program owns this connection

Connection States Explained

StateWhat It MeansLISTENINGPort is open and waiting for connections (like a door that's unlocked)ESTABLISHEDActive connection — data is flowing right nowTIME_WAITConnection just closed, waiting to fully terminateCLOSE_WAITRemote side closed, your side hasn't yetSYN_SENTTrying to connect (handshake started)

StateWhat It Means

State

What It Means

LISTENINGPort is open and waiting for connections (like a door that's unlocked)

LISTENING

Port is open and waiting for connections (like a door that's unlocked)

ESTABLISHEDActive connection — data is flowing right now

ESTABLISHED

Active connection — data is flowing right now

TIME_WAITConnection just closed, waiting to fully terminate

TIME_WAIT

Connection just closed, waiting to fully terminate

CLOSE_WAITRemote side closed, your side hasn't yet

CLOSE_WAIT

Remote side closed, your side hasn't yet

SYN_SENTTrying to connect (handshake started)

SYN_SENT

Trying to connect (handshake started)

Spotting Suspicious Connections

Look at the sample output above. Can you spot the suspicious one?

TCP    10.0.5.22:50100        185.220.101.45:4444    ESTABLISHED     6660

Why this is suspicious:

- 185.220.101.45 — This is a known Tor exit node (attacker IP)

- Port 4444 — This is the default port for Metasploit reverse shell (hacking tool)

- ESTABLISHED — The connection is ACTIVE right now

- PID 6660 — We need to find out what program this is

How to Find What Program Is Making the Connection

tasklist | find "6660"

Output:

evil_malware.exe          6660 Console    1     15,232 K

Now you know: Process ID 6660 is "evil_malware.exe" connecting to an attacker's server on port 4444. Kill it and investigate!

Now you know: Process ID 6660 is "evil_malware.exe" connecting to an attacker's server on port 4444. Kill it and investigate!

How Netstat Helps in Incident Response (Flow)

Alert: "Suspicious outbound connection detected"
     |
     | Step 1: Run netstat -ano on the machine
     v
Find suspicious connection (weird IP, weird port)
     |
     | Step 2: Note the PID (Process ID)
     v
Run: tasklist | find "PID"
     |
     | Step 3: Identify the program
     v
Is it legitimate? (chrome.exe, outlook.exe = probably fine)
Is it suspicious? (svchost.exe connecting to port 4444 = BAD)
     |
     | Step 4: If malicious
     v
Kill the process --> Isolate the machine --> Investigate further

Common Suspicious Ports to Watch For

PortWhat It Usually Means4444Metasploit default (hacking tool)5555Android Debug Bridge (if not expected)1337"Leet" — often used by hackers for fun8080Web proxy — could be C2 communication6667IRC — old school C2 channel443 to unknown IPsCould be encrypted C2 hiding in HTTPS trafficAny port > 49000 to externalUnusual — investigate what program is doing this

PortWhat It Usually Means

Port

What It Usually Means

4444Metasploit default (hacking tool)

4444

Metasploit default (hacking tool)

5555Android Debug Bridge (if not expected)

5555

Android Debug Bridge (if not expected)

1337"Leet" — often used by hackers for fun

1337

"Leet" — often used by hackers for fun

8080Web proxy — could be C2 communication

8080

Web proxy — could be C2 communication

6667IRC — old school C2 channel

6667

IRC — old school C2 channel

443 to unknown IPsCould be encrypted C2 hiding in HTTPS traffic

443 to unknown IPs

Could be encrypted C2 hiding in HTTPS traffic

Any port > 49000 to externalUnusual — investigate what program is doing this

Any port > 49000 to external

Unusual — investigate what program is doing this

Quick Comparison — When to Use Which Tool

I Want To...Use This CommandCheck if a server is alivepingSee my own IP address and network infoipconfigFind what IP a domain points tonslookupSee the path my traffic takes to reach a servertracertSee what's currently connected to my computernetstat

I Want To...Use This Command

I Want To...

Use This Command

Check if a server is aliveping

Check if a server is alive

ping

See my own IP address and network infoipconfig

See my own IP address and network info

ipconfig

Find what IP a domain points tonslookup

Find what IP a domain points to

nslookup

See the path my traffic takes to reach a servertracert

See the path my traffic takes to reach a server

tracert

See what's currently connected to my computernetstat

See what's currently connected to my computer

netstat

All Commands Cheat Sheet

# Am I connected to the network?
ipconfig

# Is the server alive?
ping 10.0.5.50

# What IP does this domain resolve to?
nslookup suspicious-domain.com

# What path does my traffic take?
tracert google.com

# What connections are active on this machine?
netstat -ano

# What program is using a specific connection?
tasklist | find "PID_NUMBER"

# Clear DNS cache after blocking a domain
ipconfig /flushdns

# Continuous ping (monitor if server stays up)
ping 10.0.5.50 -t

How These Tools Work Together in an Investigation

Scenario: User reports their computer is slow and acting weird.

Scenario: User reports their computer is slow and acting weird.

Step 1: ipconfig
   --> Check: Does the machine have the right IP? Right DNS server?
   --> Result: DNS is pointing to 185.220.101.45 (NOT our DNS server!)
   --> Conclusion: DNS might be hijacked!

Step 2: nslookup portal.cyberdx.net
   --> Check: Is our company portal resolving to the right IP?
   --> Result: It's resolving to 91.240.118.72 (wrong IP!)
   --> Conclusion: DNS IS hijacked — user is being sent to fake sites

Step 3: netstat -ano
   --> Check: What connections are active?
   --> Result: PID 6660 is connected to 185.220.101.45:4444
   --> Conclusion: Malware is running and talking to attacker

Step 4: ping 10.0.1.10 (real DNS server)
   --> Check: Can we reach our real DNS?
   --> Result: Request timed out
   --> Conclusion: Something is blocking access to real DNS

Step 5: tracert 10.0.1.10
   --> Check: Where is the traffic being blocked?
   --> Result: Traffic stops at hop 2 (10.0.5.1 — the gateway)
   --> Conclusion: Gateway/router might be compromised too

FINAL ACTION: Isolate the machine, escalate to P1, investigate the gateway

Practice Challenges

Challenge 1

You run netstat -ano and see:

TCP    10.0.5.22:51000    91.240.118.72:443    ESTABLISHED    3344

Is this suspicious? How would you investigate?

Answer: Potentially suspicious. Check what PID 3344 is (tasklist | find "3344"). Look up 91.240.118.72 on VirusTotal. If it's a known C2 server and the process is unusual — it's malware.

Challenge 2

You run nslookup microsoft.com and get:

Name:    microsoft.com
Address:  185.220.101.45

Is this normal?

Answer: NO! Microsoft.com should NOT resolve to 185.220.101.45 (a Tor exit node). This means DNS is poisoned or hijacked. The user is being redirected to a fake Microsoft site. Investigate immediately.

Challenge 3

You run tracert internal-server.cyberdx.net and see:

  1     1 ms    10.0.5.1
  2     *       Request timed out.
  3     *       Request timed out.

What does this tell you?

Answer: Traffic is being blocked after your gateway (10.0.5.1). Either a firewall rule is blocking it, the server is down, or there's a network issue between your subnet and the server's subnet. Check firewall rules and server status.

