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

```cmd
ping ping cyberdx.net
ping 192.168.1.1
ping 10.0.5.22 -t        (continuous ping — keeps going until you press Ctrl+C)
ping 10.0.5.22 -n 10     (send exactly 10 pings)
Sample Output
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
How to Read the Output

| Field | What It Means |
|-------|---------------|
| **Reply from 13.248.243.5** | The server responded, which means the host is alive and reachable. |
| **bytes=32** | Size of the ICMP packet that was sent. |
| **time=22ms** | Round-trip time for the packet. Lower values indicate a faster connection. |
| **TTL=244** | **Time To Live**. Indicates how many network hops remain before the packet is discarded. |
| **Packets Lost = 0** | All packets were received successfully, indicating a healthy connection. |

View more
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
     |     the round-trip time (12ms)        |
     |                                       |

If no reply comes back = host is down or blocked
Common Ping Errors and What They Mean

| Error | Meaning |
|-------|---------|
| **Request timed out** | The target did not respond. The host may be offline, unreachable, or blocked by a firewall. |
| **Destination host unreachable** | Your system could not find a route to the destination IP address. This usually indicates a network routing or gateway issue. |
| **TTL expired in transit** | The packet exceeded its maximum number of allowed hops before reaching the destination. This can indicate a routing loop or a very long network path. |
| **General failure** | A problem exists on the local computer, such as a disabled network adapter, incorrect network configuration, or missing network connectivity. |

2. IPCONFIG

What Does It Do?
Shows your computer's **network configuration** - your IP address, subnet mask, default gateway, and DNS servers. Think of it as your computer's "network ID card."

Real-life analogy: Like checking your home address, zip code, and which road leads to the highway. You need to know YOUR address before you can troubleshoot why you can't reach someone else.

When Do SOC Analysts Use It?
1. Check if a machine has the correct IP address
2. Verify DNS settings during a DNS poisoning investigation
3. See if DHCP assigned an unexpected IP (rogue DHCP server?)
4. Flush DNS cache after blocking a malicious domain
5. Identify which network adapter is active


How to Use It
cmd
| Command | Purpose |
|---|---|
| `ipconfig` | Displays basic IP configuration details. |
| `ipconfig /all` | Displays complete network configuration including DNS, DHCP, gateway, and adapter details. |
| `ipconfig /flushdns` | Clears the local DNS resolver cache. |
| `ipconfig /release` | Releases the current DHCP IP address. |
| `ipconfig /renew` | Requests a new IP address from DHCP. |


### Sample Output — `ipconfig`

```cmd
C:\Users\Analyst>ipconfig

Windows IP Configuration

Ethernet adapter Ethernet:

   Connection-specific DNS Suffix  . : cyberdx.net      <-- Company DNS suffix (optional)
   IPv4 Address. . . . . . . . . . . : 10.0.5.22        <-- Your computer's IP address
   Subnet Mask . . . . . . . . . . . : 255.255.255.0    <-- Defines your local network
   Default Gateway . . . . . . . . . : 10.0.5.1         <-- Router used to reach other networks

Wireless LAN adapter Wi-Fi:

   Connection-specific DNS Suffix  . : cyberdx.net
   IPv6 Address. . . . . . . . . . . : 2001:db8:abcd:100::22   <-- Example IPv6 address
   Link-local IPv6 Address . . . . . : fe80::1c2d:3e4f:5a6b:7c8d%12  <-- Local IPv6 only
   IPv4 Address. . . . . . . . . . . : 10.0.5.25        <-- Wi-Fi IP address
   Subnet Mask . . . . . . . . . . . : 255.255.255.0
   Default Gateway . . . . . . . . . : 10.0.5.1
```

## How to Read the Output

| Field | What It Means |
|-------|---------------|
| **Connection-specific DNS Suffix (cyberdx.net)** | The DNS domain assigned to your network. Common in corporate environments. |
| **IPv4 Address (10.0.5.22)** | Your computer's unique address on the local network. Other devices use this to communicate with your system. |
| **Subnet Mask (255.255.255.0)** | Defines which devices are on the same local network and which require a router to communicate. |
| **Default Gateway (10.0.5.1)** | The router that forwards traffic from your network to other networks or the Internet. |
| **IPv6 Address (2001:db8:abcd:100::22)** | Your computer's globally unique IPv6 address used on IPv6-enabled networks. |
| **Link-local IPv6 Address (fe80::...)** | A local-only IPv6 address used for communication within the same network segment. It is not routable over the Internet. |

## How `ipconfig` Relates to Your Network

```text
                  Your Computer
                    (10.0.5.22)
                         |
                         | Sends traffic to...
                         v
           Default Gateway / Router
                  (10.0.5.1)
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
            |
            +-------------------------------+
                                            |
                                            v
                              Response returns to
                              Your Computer (10.0.5.22)
```

## Security Use Cases

Common security investigation scenarios and the commands used to validate network configuration issues.

| Scenario | What to Check |
|---|---|
| Suspected DNS poisoning | Run `ipconfig /all` and verify DNS Servers. Check if DNS entries are legitimate or pointing to an unknown/attacker-controlled server. |
| Machine got a weird IP address | Run `ipconfig` or `ipconfig /all`. Verify IPv4 address, subnet, gateway, and check for possible rogue DHCP assignment. |
| Blocked a malicious domain but user can still reach it | Run `ipconfig /flushdns` to clear cached DNS entries and force a fresh DNS lookup. |
| Investigating lateral movement | Run `ipconfig /all` on the compromised host. Identify the network, subnet, gateway, and DNS configuration. |
| User cannot access internal resources | Check IP configuration, subnet alignment, gateway, and DNS resolution using `ipconfig /all`. |
| Suspected rogue DHCP server | Run `ipconfig /all` and verify the DHCP Server entry matches the approved network infrastructure. |
| Malware communicating with external infrastructure | Review DNS settings and network configuration using `ipconfig /all`. Correlate with network logs and endpoint telemetry. |
| Incident response host identification | Collect IP address, subnet mask, gateway, DNS servers, and DHCP information using `ipconfig /all`. |


# 3. NSLOOKUP

## What Does It Do?

`nslookup` (Name Server Lookup) Looks up DNS records and translates a domain name (like google.com) to an IP address, or vice versa. DNS is like the phone book of the internet.

Real-life analogy: You know your friend's name but not their phone number. You look them up in your contacts (DNS) to find their number (IP address).

SOC analysts use `nslookup` to investigate how a domain name resolves, identify suspicious IP addresses, validate DNS configurations, and troubleshoot DNS-related security incidents.

When Do SOC Analysts Use It?
1. Check if a suspicious domain resolves to a known malicious IP
2. Verify DNS is working correctly during an outage
3. Investigate phishing — where does the fake domain actually point?
4. Check if a domain was recently registered (new = suspicious)
5. Verify internal DNS records are correct


# How To Use NSLOOKUP

## Basic DNS Lookup

Find the IP address associated with a domain.

```cmd
C:\Users\Analyst>nslookup cyberdx.net

How to Use It

| Command | Purpose | SOC Use Case |
|---|---|---|
| `nslookup cyberdx.net` | Basic DNS lookup to find domain IP addresses | Verify domain resolution and investigate suspicious domains |
| `nslookup 13.248.243.5` | Reverse DNS lookup (IP address to hostname) | Identify ownership or associated domains of suspicious IPs |
| `nslookup -type=MX cyberdx.net` | Query mail exchange records | Identify mail servers during email security investigations |
| `nslookup -type=TXT cyberdx.net` | Query TXT records | Check SPF, DKIM, and other domain verification records |
| `nslookup -type=NS cyberdx.net` | Query name server records | Identify authoritative DNS servers |

Sample Output — Normal Lookup
C:\Users\Analyst>nslookup cyberdx.net

Server:  cdns01.comcast.net
Address:  75.75.75.75

Non-authoritative answer:
Name:    cyberdx.net
Addresses:  13.248.243.5
          76.223.105.230

Sample Output - Suspicious Domain

C:\Users\Analyst>nslookup totally-legit-microsoft-login.com

Server:  corp-dns-01.cyberdx.net
Address:  10.0.1.10

Non-authoritative answer:
Name:    totally-legit-microsoft-login.com
Address:  185.220.101.45
Red flag! This "Microsoft login" page is hosted on IP 185.220.101.45 — which is a known Tor exit node. This is a phishing site!

How to Read the Output

| Field | What It Means |
|---|---|
| `Server: corp-dns-01` | The DNS server that answered your query |
| `Address: 10.0.1.10` | The IP address of the DNS server |
| `Non-authoritative answer` | The response came from a DNS resolver cache and not directly from the domain's authoritative DNS server |
| `Name: totally-legit-microsoft-login.com` | The domain name you queried |
| `Addresses: 185.220.101.45` | The IP address(es) that the domain resolves to |


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

| Scenario | Command | What You're Looking For |
|---|---|---|
| Phishing email has suspicious link | `nslookup suspicious-domain.com` | Check if the domain resolves to a known malicious IP address |
| User visited a malware site | `nslookup malware-site.com` | Identify the IP address hosting the malicious content |
| DNS poisoning suspected | `nslookup company.com` | Verify if the domain resolves to the correct IP address or an unexpected IP |
| Email spoofing investigation | `nslookup -type=TXT company.com` | Check SPF, DKIM, and other TXT records used for email authentication |




4. TRACERT (Traceroute)

What Does It Do?
Shows the exact path your network traffic takes to reach a destination — every router (hop) along the way. Like GPS showing every turn on your route.

Real-life analogy: If ping is "Can I reach my friend's house?" then tracert is "Show me every street and intersection I pass through to get there."

When Do SOC Analysts Use It?
1. Find WHERE in the network path a connection is failing
2. Identify if traffic is being routed through unexpected countries (man-in-the-middle?)
3. Troubleshoot why a server is slow (which hop has high latency?)
4. Verify traffic is going through the correct firewall/proxy
5. Investigate if traffic is being redirected (BGP hijacking)


How to Use It
| cmd |
|---|---|---|
| tracert google.com | (trace route to google) |
| tracert 10.0.5.50 | (trace route to internal server) |
| tracert -d google.com | (skip DNS resolution — faster) |

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
Table



Column


What It Means


Hop number (1, 2, 3...)	Each router/device the packet passes through
Time (1 ms, 3 ms, 10 ms)	How long it took to reach that hop (3 measurements)
IP Address	The router's IP at that hop
* * * Request timed out	That router doesn't respond to tracert (firewall blocking — not necessarily bad)
View more
What Bad Output Looks Like
C:\Users\Analyst>tracert internal-server.cyberdx.net

  1     1 ms     1 ms     1 ms   10.0.5.1
  2     3 ms     2 ms     3 ms   10.0.1.1
  3     *        *        *      Request timed out.
  4     *        *        *      Request timed out.
  5     *        *        *      Request timed out.
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
Table



Scenario


What to Look For


Traffic going through unexpected country	Hops showing IPs from countries you don't expect
Connection to server is slow	Which hop has a big jump in latency? That's the bottleneck
Firewall rule verification	After blocking, tracert should stop at the firewall hop
Man-in-the-middle suspected	Extra unexpected hops that shouldn't be there
View more
5. NETSTAT
What Does It Do?
Shows all active network connections on your computer — what's connected, what's listening, and which programs are talking to the internet.

Real-life analogy: Like checking your phone's screen time — it shows you every app that's currently using the internet and who it's talking to.

When Do SOC Analysts Use It?
Check if malware is connecting to a Command & Control (C2) server
See what ports are open on a compromised machine
Identify suspicious outbound connections
Verify if a service is running and listening
Find which process is making a suspicious connection
How to Use It
cmd





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
Table



Column


What It Means


Proto	Protocol — TCP or UDP
Local Address	Your computer's IP and port
Foreign Address	The remote server's IP and port you're connected to
State	Connection status (see below)
PID	Process ID — which program owns this connection
View more
Connection States Explained
Table



State


What It Means


LISTENING	Port is open and waiting for connections (like a door that's unlocked)
ESTABLISHED	Active connection — data is flowing right now
TIME_WAIT	Connection just closed, waiting to fully terminate
CLOSE_WAIT	Remote side closed, your side hasn't yet
SYN_SENT	Trying to connect (handshake started)
View more
Spotting Suspicious Connections
Look at the sample output above. Can you spot the suspicious one?

TCP    10.0.5.22:50100        185.220.101.45:4444    ESTABLISHED     6660
Why this is suspicious:

185.220.101.45 — This is a known Tor exit node (attacker IP)
Port 4444 — This is the default port for Metasploit reverse shell (hacking tool)
ESTABLISHED — The connection is ACTIVE right now
PID 6660 — We need to find out what program this is
How to Find What Program Is Making the Connection
cmd





tasklist | find "6660"
Output:

evil_malware.exe          6660 Console    1     15,232 K
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
Table



Port


What It Usually Means


4444	Metasploit default (hacking tool)
5555	Android Debug Bridge (if not expected)
1337	"Leet" — often used by hackers for fun
8080	Web proxy — could be C2 communication
6667	IRC — old school C2 channel
443 to unknown IPs	Could be encrypted C2 hiding in HTTPS traffic
Any port > 49000 to external	Unusual — investigate what program is doing this
View more
Quick Comparison — When to Use Which Tool
Table



I Want To...


Use This Command


Check if a server is alive	ping
See my own IP address and network info	ipconfig
Find what IP a domain points to	nslookup
See the path my traffic takes to reach a server	tracert
See what's currently connected to my computer	netstat
View more
All Commands Cheat Sheet
cmd





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
Challenge 1:
You run netstat -ano and see:

TCP    10.0.5.22:51000    91.240.118.72:443    ESTABLISHED    3344
Is this suspicious? How would you investigate?
Answer: Potentially suspicious. Check what PID 3344 is (tasklist | find "3344"). Look up 91.240.118.72 on VirusTotal. If it's a known C2 server and the process is unusual — it's malware.

Challenge 2:
You run nslookup microsoft.com and get:

Name:    microsoft.com
Address:  185.220.101.45
Is this normal?
Answer: NO! Microsoft.com should NOT resolve to 185.220.101.45 (a Tor exit node). This means DNS is poisoned or hijacked. The user is being redirected to a fake Microsoft site. Investigate immediately.

Challenge 3:
You run tracert internal-server.cyberdx.net and see:

  1     1 ms    10.0.5.1
  2     *       Request timed out.
  3     *       Request timed out.
What does this tell you?
Answer: Traffic is being blocked after your gateway (10.0.5.1). Either a firewall rule is blocking it, the server is down, or there's a network issue between your subnet and the server's subnet. Check firewall rules and server status.