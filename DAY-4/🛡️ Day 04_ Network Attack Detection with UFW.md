# 🛡️ Day 04: Network Attack Detection with UFW

## 🎯 What You Will Learn

Today we learn how a SOC Analyst can detect **network reconnaissance and port scanning** using Linux firewall logs.

You will learn:

- What a port scan is
- What Nmap is
- What UFW does
- How firewall logs record blocked traffic
- How to identify a possible port scan
- How to investigate source IPs and destination ports

---

# 1. What is Network Reconnaissance?

Before attacking a system, an attacker may first try to understand what is available.

They may ask:

> Which ports are open?

> Which services are running?

> Which machines are reachable?

This activity is called **reconnaissance**.

A simple example:

```text
Attacker
   ↓
Target
   ↓
Port 22   → SSH
Port 80   → HTTP
Port 443  → HTTPS
Port 3389 → RDP
```

The attacker is trying to discover which services might be accessible.

---

# 2. What is a Port?

A **port** is a logical communication endpoint used by network services.

Some common examples:

| Port | Common Service |
|---:|---|
| 22 | SSH |
| 53 | DNS |
| 80 | HTTP |
| 443 | HTTPS |
| 3389 | RDP |

Think of a computer as a building.

```text
Computer = Building

Port 22  = Door for SSH
Port 80  = Door for HTTP
Port 443 = Door for HTTPS
```

A port scan is like checking many doors to see which ones respond.

---

# 3. What is a Port Scan?

A **port scan** checks which ports on a target system are accessible.

Example:

```text
Attacker
   ↓
Target
   ├── Port 22  → BLOCKED
   ├── Port 80  → OPEN
   ├── Port 443 → BLOCKED
   └── Port 8080 → BLOCKED
```

Attackers may perform this reconnaissance before attempting exploitation.

### Important

A port scan does **not automatically mean the system was compromised**.

It means someone or something is checking network availability.

---

# 4. What is Nmap?

**Nmap** is a network discovery and port-scanning tool.

Security professionals use it for legitimate tasks such as:

- Network discovery
- Security testing
- Service discovery
- Troubleshooting

Attackers can also abuse it for reconnaissance.

Common Nmap options:

| Command | Purpose |
|---|---|
| `nmap -sS` | SYN scan |
| `nmap -sT` | TCP connect scan |
| `nmap -sU` | UDP scan |
| `nmap -sn` | Host discovery |

For this lab, we will use a simple port scan.

---

# 5. What is UFW?

**UFW stands for Uncomplicated Firewall.**

It provides an easier way to manage firewall rules on Linux systems.

The firewall can:

```text
Allow traffic
      OR
Block traffic
```

Example:

```text
Kali
  ↓
Port 22
  ↓
Ubuntu Firewall
  ↓
BLOCKED
```

The firewall can also record traffic in logs.

---

# 6. Why Are Firewall Logs Important?

Imagine a firewall blocks hundreds of connections.

Without logs, the SOC Analyst may not know:

```text
Who sent the traffic?
Which port was targeted?
When did it happen?
What protocol was used?
```

Firewall logs provide this evidence.

For UFW, a commonly used log file is:

```text
/var/log/ufw.log
```

---

# 🧪 7. Day 04 Lab

## Lab Setup

Use two machines that you own or are authorized to test:

```text
Kali Linux
   ↓
Attacker / Scanner

Ubuntu Linux
   ↓
Victim / Target
```

### ⚠️ Safety

Only scan systems that you own or have explicit permission to test.

---

# 8. Step 1: Enable UFW

On Ubuntu:

```bash
sudo apt install ufw
```

Check the current status:

```bash
sudo ufw status
```

Enable the firewall:

```bash
sudo ufw enable
```

Enable detailed logging:

```bash
sudo ufw logging high
```

Check again:

```bash
sudo ufw status
```

---

# 9. Step 2: Generate Scan Traffic

Find the IP address of the Ubuntu machine:

```bash
ip addr
```

Suppose the Ubuntu IP is:

```text
192.168.1.20
```

From Kali, scan the target:

```bash
nmap -p80 192.168.1.20
```

This checks port 80.

For a wider lab scan:

```bash
nmap -p1-100 192.168.1.20
```

This checks ports 1 through 100.

---

# 10. Step 3: Monitor UFW Logs

On Ubuntu:

```bash
sudo tail -f /var/log/ufw.log
```

You may see entries similar to:

```text
SRC=192.168.1.10
DST=192.168.1.20
DPT=80
PROTO=TCP
```

The exact log format can vary depending on the system and configuration.

---

# 11. Understanding UFW Log Fields

Important fields include:

```text
SRC
DST
DPT
PROTO
```

### SRC

Source IP.

```text
SRC=192.168.1.10
```

Who sent the traffic?

### DST

Destination IP.

```text
DST=192.168.1.20
```

Which system received the traffic?

### DPT

Destination port.

```text
DPT=80
```

Which port was targeted?

### PROTO

Network protocol.

```text
PROTO=TCP
```

Which protocol was used?

---

# 12. The SOC Investigation Question

Don't just look at:

```text
DPT=80
```

Ask:

```text
WHO?
  ↓
SRC

WHERE?
  ↓
DST

WHAT?
  ↓
DPT / PROTO

WHEN?
  ↓
Timestamp

HOW MANY?
  ↓
Connection count

WHAT PATTERN?
  ↓
Single connection or many ports?
```

This turns a raw firewall log into an investigation.

---

# 🚨 13. Real SOC Scenario

Imagine the firewall logs show:

```text
10:20  SRC=10.0.0.25  DPT=21
10:20  SRC=10.0.0.25  DPT=22
10:20  SRC=10.0.0.25  DPT=23
10:20  SRC=10.0.0.25  DPT=25
10:20  SRC=10.0.0.25  DPT=53
10:21  SRC=10.0.0.25  DPT=80
10:21  SRC=10.0.0.25  DPT=443
```

Notice the pattern.

```text
Same Source IP
      +
Many Destination Ports
      +
Short Time Period
```

This is a strong **port-scanning/reconnaissance indicator**.

---

# 14. Single Connection vs Port Scan

### Normal-looking traffic

```text
10:30
10.0.0.50 → Port 443
```

One connection to a web server may be completely normal.

### Possible scanning

```text
10:30
10.0.0.25 → Port 21
10.0.0.25 → Port 22
10.0.0.25 → Port 23
10.0.0.25 → Port 25
10.0.0.25 → Port 53
10.0.0.25 → Port 80
```

Many different ports targeted quickly from one source is much more interesting.

---

# 15. Blocked Traffic Does Not Mean Attack

This is another important SOC lesson.

Suppose the log says:

```text
BLOCK
SRC=10.0.0.25
DPT=22
```

This means the firewall blocked the traffic.

It does **not** automatically mean:

> "An attacker tried to hack the server."

It could be:

- A legitimate security scan
- Network troubleshooting
- A monitoring system
- A misconfigured application
- An authorized penetration test
- Malicious reconnaissance

Again:

> **Context determines whether activity is suspicious.**

---

# 16. Detecting a Port Scan

A SOC Analyst can look for patterns such as:

```text
One Source IP
      ↓
Many Destination Ports
      ↓
Short Time Period
      ↓
Repeated Connections
      ↓
Firewall Blocks
      ↓
Possible Reconnaissance
```

This becomes stronger when combined with other evidence.

For example:

```text
Port Scan
   ↓
Web Server Logs
   ↓
Suspicious Requests
   ↓
Authentication Attempts
   ↓
Possible Exploitation
```

Now the investigation becomes more serious.

---

# 17. SOC Investigation Workflow

Use this workflow:

```text
Firewall Log
     ↓
Identify Source IP
     ↓
Identify Destination IP
     ↓
Check Destination Ports
     ↓
Check Protocol
     ↓
Check Timestamp
     ↓
Count Attempts
     ↓
Look for Scanning Pattern
     ↓
Correlate Other Logs
     ↓
Benign / Suspicious
     ↓
Document Finding
```

---

# 🧪 18. Useful Linux Commands

### Check UFW status

```bash
sudo ufw status
```

### Enable logging

```bash
sudo ufw logging on
```

### Watch logs live

```bash
sudo tail -f /var/log/ufw.log
```

### Search for an IP

```bash
sudo grep "192.168.1.10" /var/log/ufw.log
```

Replace the IP with your actual Kali machine IP.

### Search for a port

```bash
sudo grep "DPT=80" /var/log/ufw.log
```

These commands are useful for basic log investigation.

---

# 🎯 19. Day 04 SOC Challenge

After completing the scan, record:

### Source

```text
Attacker IP:
```

### Destination

```text
Victim IP:
```

### Ports

```text
Destination Ports:
```

### Protocol

```text
Protocol:
```

### Time

```text
First Attempt:
Last Attempt:
```

### Volume

```text
Number of Attempts:
```

### Firewall Result

```text
Allowed / Blocked:
```

### Detection

Answer:

> Does the traffic look like a port scan? Why?

---

# 🔎 20. Real SOC Example

Imagine a SOC receives this activity from an unknown workstation:

```text
09:15:01 → Port 21
09:15:01 → Port 22
09:15:02 → Port 23
09:15:02 → Port 25
09:15:02 → Port 53
09:15:03 → Port 80
09:15:03 → Port 110
09:15:04 → Port 443
```

The analyst notices:

```text
Same source
     +
Many ports
     +
Very short time period
```

### Initial Assessment

```text
Possible Network Reconnaissance
```

### Next Steps

The analyst should investigate:

```text
Who owns the source IP?
       ↓
Is scanning authorized?
       ↓
Were any ports actually open?
       ↓
Were there follow-up connections?
       ↓
Were suspicious application requests made?
```

This prevents the SOC from immediately declaring an attack without evidence.

---

# 🧠 Day 04 Mindset

Don't think:

> "Nmap = Attack."

Think:

> "This source is probing multiple ports. Why is it doing that?"

Then investigate:

```text
WHO?
 ↓
WHERE?
 ↓
WHICH PORT?
 ↓
HOW MANY?
 ↓
HOW FAST?
 ↓
ALLOWED OR BLOCKED?
 ↓
WHAT HAPPENED NEXT?
```

That is the SOC mindset.

---

# 📌 Day 04 Takeaways

You should now understand:

- What network reconnaissance is
- What a port scan is
- What Nmap does
- What UFW is
- Where UFW logs are stored
- How to read `SRC`, `DST`, `DPT`, and `PROTO`
- How to identify repeated connection attempts
- How multiple destination ports can indicate scanning
- Why blocked traffic does not automatically mean an attack
- How to correlate firewall activity with other logs

---

# 💼 Portfolio Skills

```text
Linux Firewall Analysis
UFW
Nmap
Network Reconnaissance Detection
Firewall Log Analysis
IP and Port Analysis
Basic Threat Detection
SOC Investigation
Event Correlation
Security Documentation
```

---

# 🧠 One-Line Lesson

> **A port scan is not proof of compromise. It is a reconnaissance signal that needs investigation and context.**