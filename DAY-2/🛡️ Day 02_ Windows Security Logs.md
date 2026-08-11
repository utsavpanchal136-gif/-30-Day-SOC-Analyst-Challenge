# 🛡️ Day 02: Windows Security Logs

## 🎯 What You Will Learn

Today we learn how SOC Analysts investigate **Windows login activity**.

You will learn:

- What Windows Security Logs are
- Important authentication Event IDs
- How to detect failed logins
- How to investigate brute-force indicators
- How to correlate failed and successful logins

---

# 1. What Are Windows Security Logs?

Windows Security Logs record important security-related activities on a Windows computer.

For example:

```text
User logs in
       ↓
Windows creates a Security Event
       ↓
SOC Analyst investigates the event
```

These logs can help answer:

> Who logged in?  
> When did they log in?  
> Was the login successful?  
> Where did it come from?  
> What privileges were assigned?

---

# 2. Why Are Security Logs Important?

Attackers often need to authenticate to systems.

They may:

- Guess passwords
- Steal credentials
- Use compromised accounts
- Create new accounts
- Add users to privileged groups
- Access systems remotely

Windows Security Logs can provide evidence of these activities.

### Real-Life Cyber Scenario

Imagine an attacker knows that a company's administrator account is called:

```text
Administrator
```

They start trying different passwords:

```text
Administrator + Password1
       ↓
FAILED

Administrator + Welcome123
       ↓
FAILED

Administrator + Summer2026
       ↓
FAILED

Administrator + CorrectPassword
       ↓
SUCCESS
```

Windows can record these authentication attempts.

A SOC Analyst can use those events to investigate the activity.

---

# 3. Important Windows Security Event IDs

| Event ID | Meaning | SOC Use |
|---|---|---|
| **4624** | Successful Logon | Investigate unusual successful access |
| **4625** | Failed Logon | Detect suspicious authentication attempts |
| **4740** | Account Locked Out | Investigate repeated failed logins |
| **4732** | User added to local security group | Detect possible privilege changes |
| **4672** | Special privileges assigned | Investigate privileged access |

### 🧠 Easy Memory Trick

```text
4624 = LOGIN SUCCESS ✅

4625 = LOGIN FAILED ❌

4740 = ACCOUNT LOCKED 🔒

4732 = GROUP CHANGED 👥

4672 = SPECIAL PRIVILEGES ⚡
```

---

# 4. Event ID 4624

```text
4624 = Successful Logon
```

This means Windows recorded a successful authentication.

Example:

```text
User: administrator
Host: SERVER-01
Time: 10:30
Result: Successful Logon
```

A successful login is **not automatically suspicious**.

For example:

```text
Employee → Company Laptop → Normal working hours
```

This is probably normal.

But:

```text
Employee → Server → 3:00 AM → Unusual location
```

may deserve investigation.

### SOC Question

> Was this successful login expected?

---

# 5. Event ID 4625

```text
4625 = Failed Logon
```

This means an authentication attempt failed.

Example:

```text
Account: administrator
Time: 10:32
Result: Failed Logon
Reason: Incorrect password
```

One failed login is usually not a big deal.

People make mistakes.

But repeated failures can become interesting.

---

# 6. Event ID 4740

```text
4740 = Account Locked Out
```

Windows can lock an account after too many failed authentication attempts, depending on the organization's security policy.

Example:

```text
4625
4625
4625
4625
4625
      ↓
4740
      ↓
Account Locked
```

This could indicate:

- Someone repeatedly entering the wrong password
- A forgotten password
- A misconfigured application
- A brute-force attempt

Again, **context matters**.

---

# 7. Event ID 4732

```text
4732 = Member Added to a Security-Enabled Local Group
```

This can be important because adding a user to a privileged group can increase their access.

Example:

```text
Normal User
     ↓
Added to Administrators group
     ↓
Higher privileges
```

If an attacker gains administrator-level access, they may have much more control over the machine.

A SOC Analyst should investigate unexpected group changes.

---

# 8. Event ID 4672

```text
4672 = Special Privileges Assigned to New Logon
```

This event can appear when a user logs in with powerful privileges.

Example:

```text
User logs in
     ↓
Windows assigns special privileges
     ↓
4672 generated
```

This does **not automatically mean the account is compromised**.

Administrators legitimately use privileged accounts.

The analyst needs to determine whether the privileged login was expected.

---

# 9. What Should a SOC Analyst Look For?

When investigating an authentication event, ask:

### 👤 WHO?

Which account was involved?

```text
Account: administrator
```

### 🕐 WHEN?

When did it happen?

```text
Time: 02:13 AM
```

### 🌐 WHERE?

Where did the login come from?

```text
Source IP: 10.0.0.25
```

### 🔑 WHAT TYPE?

What type of logon was it?

Windows provides a **Logon Type** that helps explain how the authentication happened.

For example:

```text
Logon Type 2  → Interactive
Logon Type 3  → Network
Logon Type 10 → Remote Interactive / RDP
```

These details help the analyst understand the situation.

### ❓ IS IT NORMAL?

Ask:

> Does this activity make sense for this user and machine?

---

# 10. 🧪 Day 02 Lab: Generate a Failed Login

## Requirements

- Windows 10/11
- Event Viewer
- PowerShell or Command Prompt
- Administrator access

---

## Step 1: Create a Test User

Open Command Prompt or PowerShell as Administrator:

```cmd
net user haxuser1 TestPassword123 /add
```

This creates a local test account.

You can verify it with:

```cmd
net user haxuser1
```

---

# Step 2: Generate a Failed Authentication

Run:

```cmd
net use \\127.0.0.1\IPC$ /user:haxuser1 WrongPassword
```

The password is intentionally incorrect.

This should generate a failed authentication event.

### ⚠️ Important

`127.0.0.1` means:

```text
This computer
     ↑
     └── localhost
```

So this lab is generating a **local test event**.

It demonstrates how a failed authentication appears in Windows logs. It does not simulate an external attacker.

---

# Step 3: Open Event Viewer

Run:

```text
eventvwr.msc
```

Navigate to:

```text
Windows Logs
→ Security
```

Look for:

```text
Event ID: 4625
```

You can filter the Security log for Event ID `4625`.

---

# 11. Investigate Event ID 4625

When you find the event, look for important fields such as:

```text
Account Name
Account Domain
Logon Type
Failure Reason
Source Network Address
Timestamp
```

Record the information.

Example:

```text
Event ID: 4625
Account: haxuser1
Logon Type: 3
Failure Reason: Unknown user name or bad password
Source Address: 127.0.0.1
```

---

# 12. SOC Investigation Questions

Use these questions when analyzing the event.

### WHO?

Who attempted the login?

```text
haxuser1
```

### WHEN?

When did it happen?

```text
10:32:15
```

### WHERE?

Where did the authentication originate?

```text
127.0.0.1
```

### WHY?

Why did authentication fail?

```text
Incorrect password
```

### WHAT TYPE?

What was the Logon Type?

```text
3 = Network
```

### IS IT SUSPICIOUS?

Ask:

> Is this expected activity or something unusual?

---

# 🚨 13. Real SOC Scenario: Brute Force

Imagine a company SOC receives these events:

```text
10:01  4625  administrator  10.0.0.25
10:02  4625  administrator  10.0.0.25
10:03  4625  administrator  10.0.0.25
10:04  4625  administrator  10.0.0.25
10:05  4625  administrator  10.0.0.25
10:06  4625  administrator  10.0.0.25
```

This pattern is more suspicious than one failed login.

Why?

Because we have:

```text
Same account
      +
Repeated failures
      +
Same source
```

This could indicate a **brute-force or password-guessing attempt**.

---

# 🚨 14. Failed Login + Successful Login

Now imagine:

```text
10:01  4625  administrator  10.0.0.25
10:02  4625  administrator  10.0.0.25
10:03  4625  administrator  10.0.0.25
10:04  4625  administrator  10.0.0.25
10:05  4624  administrator  10.0.0.25
```

This is much more interesting.

The pattern is:

```text
Failed
  ↓
Failed
  ↓
Failed
  ↓
Failed
  ↓
SUCCESS
```

The SOC Analyst should investigate.

Possible questions:

- Was the account compromised?
- Was the successful login legitimate?
- Was the source IP expected?
- What happened after the login?
- Were any privileged actions performed?

---

# 15. Don't Alert on One Event

This is a very important SOC concept.

### One failed login

```text
4625
 ↓
Probably normal
```

Someone may simply have typed the wrong password.

### Many failed logins

```text
4625
4625
4625
4625
 ↓
Suspicious
```

Potential password guessing.

### Many failures + successful login

```text
4625
4625
4625
4625
 ↓
4624
 ↓
HIGHER PRIORITY INVESTIGATION
```

The exact severity still depends on the environment and context.

---

# 🧠 16. Correlation

**Correlation means connecting multiple events to understand the bigger picture.**

Instead of looking at:

```text
One 4625
```

look at:

```text
4625
  ↓
4625
  ↓
4625
  ↓
4624
  ↓
4672
  ↓
Suspicious activity
```

Now the story becomes much more interesting.

The SOC Analyst is not just reading individual logs.

They are **connecting events together**.

---

# 17. Real-Life Example

Imagine a company's finance server.

At 2:00 AM:

```text
Multiple failed logins
```

At 2:05 AM:

```text
Successful login
```

At 2:06 AM:

```text
Special privileges assigned
```

At 2:08 AM:

```text
New user added to local administrators
```

Relevant events might include:

```text
4625
 ↓
4625
 ↓
4624
 ↓
4672
 ↓
4732
```

This chain is significantly more suspicious than any single event.

A SOC Analyst should investigate the entire sequence.

---

# 🔎 SOC Investigation Workflow

Use this simple workflow:

```text
Security Event
      ↓
Identify User
      ↓
Identify Source
      ↓
Check Timestamp
      ↓
Check Logon Type
      ↓
Check Failure Reason
      ↓
Look for Related Events
      ↓
Correlate Events
      ↓
Determine Risk
      ↓
Document Finding
```

---

# 📌 Important Concepts

Remember:

```text
4624
↓
Successful Login

4625
↓
Failed Login

4740
↓
Account Lockout

4732
↓
User Added to Local Security Group

4672
↓
Special Privileges Assigned
```

And the most important SOC principle:

> **One event rarely tells the whole story. Correlate events and investigate the context.**

---

# 🎯 Day 02 Takeaways

By the end of Day 02, you should understand:

- What Windows Security Logs are
- What Event ID 4624 means
- What Event ID 4625 means
- What Event ID 4740 means
- What Event ID 4732 means
- What Event ID 4672 means
- How to investigate failed authentication
- How brute-force activity can appear in logs
- Why successful login after repeated failures matters
- How to correlate multiple security events

---

# 💼 Portfolio Skills

```text
Windows Security Log Analysis
Authentication Monitoring
Event ID Analysis
Brute-Force Detection
Event Correlation
Basic Incident Investigation
Security Documentation
```

---

# 🧠 Day 02 Mindset

Don't think:

> "4625 = Attack"

Think:

> "4625 = Failed authentication. Now I need context."

Then investigate:

```text
WHO?
WHAT?
WHEN?
WHERE?
HOW?
WHY?
WHAT HAPPENED NEXT?
```

That's the mindset of a SOC Analyst.