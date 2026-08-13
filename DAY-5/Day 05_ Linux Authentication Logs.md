# 🛡️ Day 05: Linux Authentication Logs

## 🎯 What You Will Learn

Today we learn how SOC Analysts investigate **Linux authentication logs** and detect possible **SSH brute-force attacks**.

You will learn:

- What SSH authentication is
- What an SSH brute-force attack looks like
- Where Linux authentication logs are stored
- How to search authentication logs
- How to identify repeated login failures
- How to correlate failed and successful logins
- Basic ways to protect SSH

---

# 1. What is SSH?

**SSH (Secure Shell)** is a protocol used to remotely access and manage Linux systems.

Example:

```text
Administrator
     ↓
SSH
     ↓
Linux Server
```

An administrator might connect using:

```bash
ssh username@server-ip
```

SSH is widely used for server administration.

Because SSH provides remote access, attackers often target it.

---

# 2. What is an SSH Brute-Force Attack?

A brute-force attack happens when someone repeatedly tries different passwords against an account.

Example:

```text
Attacker
   ↓
SSH Server
   ↓
Wrong Password ❌
Wrong Password ❌
Wrong Password ❌
Wrong Password ❌
Correct Password ✅
```

The attacker is trying to discover valid credentials.

---

# 3. Real-Life Cyber Scenario

Imagine a company has a Linux server with SSH enabled.

The server is exposed to the network.

An attacker discovers the server and starts trying passwords:

```text
01:10 → admin → Failed
01:10 → admin → Failed
01:11 → admin → Failed
01:11 → admin → Failed
01:11 → admin → Failed
01:12 → admin → Success
```

A SOC Analyst may notice this pattern in the Linux authentication logs.

The important question becomes:

> Was the successful login legitimate or did the attacker obtain the password?

---

# 4. Linux Authentication Logs

Linux records authentication-related activity in system logs.

### Ubuntu / Debian

```text
/var/log/auth.log
```

### RHEL / CentOS

```text
/var/log/secure
```

These logs can contain information about:

- Successful logins
- Failed logins
- SSH activity
- Authentication failures
- User activity
- Privileged access

---

# 5. What Does an Authentication Log Look Like?

Example:

```text
Failed password for testuser from 192.168.1.10 port 54321 ssh2
```

This tells us several things:

```text
Username:
testuser

Source IP:
192.168.1.10

Source Port:
54321

Protocol:
SSH
```

The timestamp is also included in the log entry.

---

# 6. What Should a SOC Analyst Look For?

When investigating SSH activity, ask:

### 👤 WHO?

Which account is being targeted?

```text
testuser
```

### 🌐 WHERE FROM?

What source IP is making the connection?

```text
192.168.1.10
```

### 🕐 WHEN?

When did the attempts happen?

```text
01:10
01:10
01:11
01:11
```

### 🔢 HOW MANY?

How many authentication failures occurred?

### ⚡ HOW FAST?

Did they happen over seconds, minutes, or hours?

### ✅ DID IT EVENTUALLY SUCCEED?

This is especially important.

---

# 7. One Failed Login vs Brute Force

A single failed login:

```text
Failed
 ↓
Probably normal
```

could simply mean someone typed the wrong password.

Repeated failures:

```text
Failed
Failed
Failed
Failed
Failed
 ↓
Suspicious
```

are more interesting.

Repeated failures from the same source against the same account in a short period can indicate password guessing.

---

# 🚨 8. Stronger Indicator: Failure → Success

Consider:

```text
10:01 → Failed
10:01 → Failed
10:02 → Failed
10:02 → Failed
10:03 → Success
```

This deserves investigation.

The SOC Analyst should ask:

```text
Who logged in?
      ↓
Where did they come from?
      ↓
Was the login expected?
      ↓
What happened after login?
```

The successful login does **not automatically prove compromise**, but it increases the importance of the investigation.

---

# 🧪 9. Day 05 Lab

## Lab Setup

Use two machines that you own or are authorized to test:

```text
Kali Linux
    ↓
SSH
    ↓
Ubuntu Server
```

Kali will generate authentication attempts.

Ubuntu will record them.

### ⚠️ Safety

Only perform this test against systems you own or have explicit permission to test.

---

# 10. Step 1: Verify SSH

On Ubuntu:

```bash
sudo systemctl status ssh
```

If SSH is not running:

```bash
sudo systemctl enable --now ssh
```

Check again:

```bash
sudo systemctl status ssh
```

You should see that the SSH service is running.

---

# 11. Step 2: Create a Test Account

For the lab, use a dedicated test account rather than a real administrator account.

Example:

```bash
sudo adduser testuser
```

Set a lab-only password when prompted.

---

# 12. Step 3: Generate Failed Logins

From your Kali machine, use your authorized lab target.

For example, Hydra can be used to generate authentication attempts:

```bash
hydra -l testuser -P passwords.txt ssh://TARGET-IP
```

Keep the password list small.

### Purpose of this lab

The goal is **not to crack an account**.

The goal is:

```text
Generate Authentication Attempts
             ↓
Create Logs
             ↓
Investigate Logs
```

---

# 13. Step 4: Find Failed SSH Attempts

On Ubuntu:

```bash
sudo grep "Failed password" /var/log/auth.log
```

You may see entries similar to:

```text
Failed password for testuser from 192.168.1.10 port 54321 ssh2
Failed password for testuser from 192.168.1.10 port 54322 ssh2
Failed password for testuser from 192.168.1.10 port 54323 ssh2
```

Now you have evidence of repeated authentication failures.

---

# 14. Watch Authentication Logs in Real Time

You can monitor the log while the lab is running:

```bash
sudo tail -f /var/log/auth.log
```

This is useful for SOC-style monitoring.

You can see events appearing as they happen.

```text
SSH Attempt
    ↓
Authentication Failure
    ↓
Log Generated
    ↓
SOC Analyst Sees Event
```

---

# 15. Search for a Specific IP

Suppose your Kali IP is:

```text
192.168.1.10
```

Search for it:

```bash
sudo grep "192.168.1.10" /var/log/auth.log
```

This helps isolate activity from one source.

---

# 16. Count Failed Attempts

You can count failed authentication entries:

```bash
sudo grep "Failed password" /var/log/auth.log | wc -l
```

You can also investigate which source IP appears repeatedly.

A useful approach is to extract the IP addresses and count them:

```bash
sudo grep "Failed password" /var/log/auth.log | awk '{print $(NF-3)}' | sort | uniq -c | sort -nr
```

Example result:

```text
25 192.168.1.10
3  192.168.1.20
1  192.168.1.30
```

This tells you that `192.168.1.10` generated the most failed authentication events.

---

# 🔎 17. SOC Investigation

Suppose you discover:

```text
25 failed SSH attempts
Source: 192.168.1.10
Target: testuser
Time: 10:20 - 10:22
```

The pattern is:

```text
Same Source
     +
Same Account
     +
Many Failures
     +
Short Time Period
```

### Initial Assessment

```text
Possible SSH Brute Force
```

But don't stop there.

Ask:

> Is `192.168.1.10` an authorized administrator?

If it is your company's vulnerability scanner or security team, the activity may be legitimate.

---

# 🚨 18. Real SOC Scenario

Imagine a production server receives:

```text
10:01 → admin → Failed → 185.x.x.x
10:01 → admin → Failed → 185.x.x.x
10:02 → admin → Failed → 185.x.x.x
10:02 → admin → Failed → 185.x.x.x
10:03 → admin → Success → 185.x.x.x
```

Then shortly afterward:

```text
10:04 → sudo command
10:05 → New SSH key added
10:06 → New user created
```

Now the investigation becomes much more serious.

Possible sequence:

```text
SSH Failures
     ↓
Successful Login
     ↓
Privileged Activity
     ↓
Account/Access Change
```

This could indicate a compromised account.

The SOC should investigate the entire sequence.

---

# 19. What Happens After Successful Login?

A successful SSH login is only the beginning of the investigation.

Check for:

```text
sudo activity
Shell commands
New users
SSH keys
File changes
Network connections
Process creation
```

The question is:

> What did the account do after authentication?

---

# 🛡️ 20. Basic SSH Defenses

Organizations can reduce SSH attack risk with controls such as:

### Use SSH Keys

Instead of relying only on passwords.

```text
SSH Key
   ↓
Stronger Authentication
```

### Disable Direct Root Login

Avoid allowing direct remote root access where appropriate.

### Restrict SSH Access

Only trusted networks or users should be allowed where possible.

### Rate Limiting / Blocking

Tools such as Fail2ban can automatically react to repeated authentication failures.

### Monitor Logs

Send authentication logs to a SIEM for centralized monitoring and alerting.

---

# 21. SOC Investigation Workflow

Use this process:

```text
Authentication Log
        ↓
Identify Source IP
        ↓
Identify Target Account
        ↓
Count Failures
        ↓
Check Time Pattern
        ↓
Check Successful Login
        ↓
Investigate Post-Login Activity
        ↓
Correlate Other Logs
        ↓
Determine Risk
        ↓
Document Finding
```

---

# 🎯 22. Day 05 SOC Challenge

After completing the lab, record:

### Source IP

```text
<source IP>
```

### Target

```text
<username>
```

### Number of Failures

```text
<number>
```

### Time Range

```text
<first timestamp> → <last timestamp>
```

### Successful Login?

```text
Yes / No
```

### Source Expected?

```text
Yes / No / Unknown
```

### What Happened After Authentication?

```text
<your investigation>
```

### Final Assessment

```text
Benign / Suspicious / Requires Investigation
```

Explain **why**.

---

# 🧠 Day 05 Mindset

Don't think:

> "Many failed logins = definitely an attack."

Think:

> "There are repeated SSH authentication failures. Who is generating them, which account is targeted, how quickly are they occurring, and did authentication eventually succeed?"

Then investigate:

```text
WHO?
 ↓
FROM WHERE?
 ↓
WHICH ACCOUNT?
 ↓
HOW MANY?
 ↓
HOW FAST?
 ↓
SUCCESS?
 ↓
WHAT HAPPENED NEXT?
```

That is the SOC mindset.

---

# 📌 Day 05 Takeaways

You should now understand:

- What SSH is
- What SSH brute force means
- Where Linux authentication logs are stored
- How to search `/var/log/auth.log`
- How to monitor authentication activity
- How to identify repeated failures
- How to identify suspicious source IPs
- Why failure → success is important
- How to investigate post-login activity
- Basic SSH security controls

---

# 💼 Portfolio Skills

```text
Linux Authentication Log Analysis
SSH Monitoring
Brute-Force Detection
Linux Command-Line Investigation
Source IP Analysis
Authentication Investigation
Event Correlation
Basic Incident Response
Security Documentation
```

---

# 🧠 One-Line Lesson

> **Repeated SSH failures are a signal. Correlating the source, timing, account, successful access, and post-login activity turns that signal into a useful detection.**