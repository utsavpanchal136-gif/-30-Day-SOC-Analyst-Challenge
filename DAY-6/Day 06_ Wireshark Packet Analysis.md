# 🛡️ Day 06: Wireshark Packet Analysis

## 🎯 What You Will Learn

Today we learn how SOC Analysts use **Wireshark** to investigate network traffic at the packet level.

You will learn:

- What packets are
- What PCAP files are
- How Wireshark works
- How to use basic display filters
- How to identify source and destination systems
- How to investigate interesting network communication
- How to recognize possible suspicious network patterns

---

# 1. What is Wireshark?

**Wireshark** is a network protocol analyzer.

It allows analysts to capture and inspect network traffic.

Think of it as a **CCTV system for network communication**.

```text
Network Traffic
      ↓
   Wireshark
      ↓
   Packets
      ↓
IPs + Ports + Protocols
      ↓
Investigation
```

A SOC Analyst can use Wireshark during:

- Incident investigation
- Threat hunting
- Malware analysis
- Network troubleshooting
- Suspicious traffic investigation

---

# 2. What is a Network Packet?

A **packet** is a small unit of data sent across a network.

For example:

```text
Computer A
    ↓
   Packet
    ↓
Computer B
```

A packet contains information that helps the network deliver it to the correct destination.

A SOC Analyst may be interested in:

```text
Source IP
Destination IP
Protocol
Source Port
Destination Port
Timestamp
```

---

# 3. What is a Protocol?

A **protocol** defines how devices communicate over a network.

Common examples:

| Protocol | Common Purpose |
|---|---|
| TCP | Reliable network communication |
| UDP | Connectionless communication |
| DNS | Domain name resolution |
| HTTP | Web communication |
| HTTPS | Encrypted web communication |
| ICMP | Network diagnostics |

Example:

```text
Computer
   ↓
DNS Request
   ↓
DNS Server
   ↓
IP Address
```

---

# 4. What is a PCAP?

**PCAP** is a file containing captured network traffic.

Common extensions:

```text
.pcap
.pcapng
```

Think of a PCAP as a **recording of network activity**.

```text
Network Activity
       ↓
Packet Capture
       ↓
PCAP File
       ↓
Wireshark
       ↓
Investigation
```

This is extremely useful when investigating an incident after the traffic has already happened.

---

# 5. Understanding Wireshark's Interface

When you open a capture, you will normally work with three main areas.

### 1. Packet List

Shows captured packets.

Example:

```text
No.   Time   Source       Destination    Protocol
1     0.001  192.168.1.5  8.8.8.8        DNS
2     0.015  192.168.1.5  1.1.1.1        DNS
3     0.021  192.168.1.5  93.x.x.x       TCP
```

### 2. Packet Details

Shows information about the selected packet.

You can inspect:

```text
Ethernet
IP
TCP/UDP
Application Protocol
```

### 3. Packet Bytes

Shows the raw packet data.

For beginners, focus mainly on **Packet List + Packet Details**.

---

# 6. Important Network Fields

When investigating a packet, understand these fields.

### Source IP

The system sending the packet.

```text
SRC = 192.168.1.10
```

### Destination IP

The system receiving the packet.

```text
DST = 8.8.8.8
```

### Source Port

The port used by the sender.

### Destination Port

The service being contacted.

Example:

```text
192.168.1.10:54321
        ↓
8.8.8.8:53
```

This could represent a DNS request.

---

# 7. Basic Wireshark Filters

Wireshark becomes much more useful when you filter traffic.

### Show HTTP traffic

```text
http
```

### Show DNS traffic

```text
dns
```

### Show TCP traffic

```text
tcp
```

### Show traffic involving an IP

```text
ip.addr == 192.168.1.10
```

### Show traffic using TCP port 80

```text
tcp.port == 80
```

### Show traffic from an IP

```text
ip.src == 192.168.1.10
```

### Show traffic going to an IP

```text
ip.dst == 192.168.1.10
```

---

# 🧪 8. Day 06 Lab

You can use either:

```text
Your own authorized network capture
```

or:

```text
A provided PCAP/PCAPNG file
```

### ⚠️ Safety

Only capture or analyze traffic that you are authorized to monitor.

---

# 9. Step 1: Open a PCAP

Open Wireshark.

Then open your:

```text
.pcap
```

or:

```text
.pcapng
```

file.

You will see many packets.

Don't try to analyze everything at once.

Start with one protocol or one IP.

---

# 10. Step 2: Start with DNS

Apply:

```text
dns
```

You should now see DNS-related packets.

Look for:

```text
Source IP
Destination IP
Query Name
Response
Timestamp
```

Example:

```text
Client
  ↓
DNS Query
  ↓
example.com
```

### SOC Question

Ask:

> Is the domain expected for this system?

---

# 11. Step 3: Investigate TCP

Apply:

```text
tcp
```

Now investigate TCP communication.

You may see:

```text
Client
   ↓
TCP connection
   ↓
Server
```

Look at:

```text
Source IP
Destination IP
Source Port
Destination Port
```

For example:

```text
192.168.1.10:51432
        ↓
93.x.x.x:443
```

This could represent an HTTPS connection.

---

# 12. Step 4: Investigate One IP

Choose an interesting IP from the capture.

For example:

```text
192.168.1.10
```

Use:

```text
ip.addr == 192.168.1.10
```

Now Wireshark will show traffic involving that IP.

This makes it easier to understand what that system was communicating with.

---

# 13. Step 5: Follow a Conversation

If you find an interesting TCP packet, right-click it and use the option to **Follow the TCP Stream**.

This can help you understand the communication as a conversation rather than looking at individual packets.

For encrypted protocols such as HTTPS, you generally won't be able to read the application content without the appropriate decryption material.

---

# 🔎 14. What Should a SOC Analyst Look For?

Don't randomly search packets.

Look for patterns.

### DNS

```text
Client
  ↓
DNS Request
  ↓
Unusual Domain
```

Questions:

- Is the domain expected?
- Is the system repeatedly requesting it?
- Does the domain look suspicious?

---

### Repeated Connections

```text
Victim
  ↓
IP A
  ↓
IP A
  ↓
IP A
  ↓
IP A
```

Repeated connections can be interesting.

But they could also be completely normal.

For example:

- Web applications
- Software updates
- Cloud services
- Time synchronization
- Monitoring systems

Context matters.

---

# 🚨 15. Possible Malware Communication

A simplified suspicious pattern might look like:

```text
Victim
   ↓
DNS Query
   ↓
Suspicious Domain
   ↓
External IP
   ↓
Repeated Connections
```

This could be a **malware communication indicator**.

In some cases, malware communicates with a command-and-control server.

However:

> **Repeated connections alone do not prove C2.**

The analyst needs additional evidence.

---

# 16. Example Investigation

Suppose you find:

```text
Source:
192.168.1.20

Destination:
203.0.113.50

Protocol:
TCP

Destination Port:
443
```

You should not immediately say:

> "This is malicious."

Instead investigate:

```text
Who owns the destination IP?
        ↓
Is the destination expected?
        ↓
What domain was contacted?
        ↓
How frequently does the connection occur?
        ↓
What process generated the traffic?
        ↓
Are there other suspicious events?
```

This is proper SOC reasoning.

---

# 17. Clear-Text Credentials

Another thing analysts may look for is **unencrypted communication**.

For example, older protocols such as:

```text
FTP
Telnet
HTTP
```

may expose information that should be protected.

If a PCAP contains credentials transmitted in clear text, that can be a serious security issue.

### Important

Do not test this against systems you do not own.

Use authorized lab traffic or a provided PCAP.

---

# 18. Useful Filters for Investigation

### DNS queries

```text
dns
```

### TCP traffic

```text
tcp
```

### UDP traffic

```text
udp
```

### HTTP

```text
http
```

### Traffic involving an IP

```text
ip.addr == 192.168.1.10
```

### Traffic from an IP

```text
ip.src == 192.168.1.10
```

### Traffic to an IP

```text
ip.dst == 192.168.1.10
```

### TCP port

```text
tcp.port == 443
```

---

# 19. SOC Investigation Workflow

Use this process:

```text
PCAP
 ↓
Identify Interesting Traffic
 ↓
Filter by Protocol/IP/Port
 ↓
Identify Source
 ↓
Identify Destination
 ↓
Inspect Packet Details
 ↓
Follow Conversation
 ↓
Look for Patterns
 ↓
Correlate With Other Evidence
 ↓
Determine Risk
 ↓
Document Finding
```

---

# 🎯 20. Day 06 SOC Challenge

Find **one interesting communication** in your PCAP.

Record:

### Source IP

```text
<IP>
```

### Destination IP

```text
<IP>
```

### Protocol

```text
<TCP / UDP / DNS / HTTP / etc.>
```

### Source Port

```text
<port>
```

### Destination Port

```text
<port>
```

### Timestamp

```text
<timestamp>
```

### Application / Service

```text
<service>
```

### Filter Used

```text
<Wireshark filter>
```

---

# 🔍 Investigation Questions

Answer:

### 1. What systems were communicating?

### 2. What protocol was being used?

### 3. What service/port was involved?

### 4. Was the communication expected?

### 5. Was there any unusual behavior?

### 6. What additional evidence would you check?

Possible sources:

```text
DNS Logs
Firewall Logs
Windows Logs
Linux Logs
EDR
Proxy Logs
SIEM
```

---

# 🚨 21. Real SOC Scenario

Imagine an employee's computer generates this sequence:

```text
10:30
DNS query
     ↓
Unusual domain

10:30
Connection
     ↓
External IP

10:31
Connection
     ↓
Same external IP

10:32
Connection
     ↓
Same external IP
```

The SOC Analyst now asks:

```text
Is the domain legitimate?
        ↓
Is the IP trusted?
        ↓
Which process created the connection?
        ↓
Does the endpoint show suspicious activity?
```

If multiple pieces of evidence point in the same direction, the investigation becomes stronger.

---

# 🧠 Day 06 Mindset

Don't think:

> "This IP looks strange, so it must be malicious."

Think:

> "This system is communicating with this destination. Why is it communicating, how often, using which protocol, and is that behavior expected?"

Then investigate:

```text
WHO?
 ↓
TALKING TO WHOM?
 ↓
USING WHAT PROTOCOL?
 ↓
WHICH PORT?
 ↓
HOW OFTEN?
 ↓
WHAT CONTENT?
 ↓
IS IT EXPECTED?
 ↓
WHAT OTHER EVIDENCE EXISTS?
```

---

# 📌 Day 06 Takeaways

You should now understand:

- What Wireshark is
- What packets are
- What protocols are
- What PCAP files are
- How to navigate Wireshark
- How to use display filters
- How to identify source and destination IPs
- How to analyze ports and protocols
- How to investigate DNS traffic
- How to investigate TCP traffic
- How to recognize potentially suspicious communication
- Why one unusual packet is not enough to confirm an attack

---

# 💼 Portfolio Skills

```text
Wireshark
PCAP Analysis
Network Traffic Analysis
Packet Investigation
IP and Port Analysis
Protocol Analysis
DNS Investigation
Basic Threat Hunting
Network Forensics
SOC Investigation
Security Documentation
```

---

# 🧠 One-Line Lesson

> **A packet is only one piece of evidence. A SOC Analyst looks at the communication pattern and context to understand what actually happened.**