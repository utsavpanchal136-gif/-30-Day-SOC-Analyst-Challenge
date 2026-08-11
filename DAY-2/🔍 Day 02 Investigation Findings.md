# 🔍 Day 02 Investigation Findings

## Event Summary

| Field | Value |
|---|---|
| Event ID | 4625 |
| Activity | Failed Logon |
| User | `haxuser1` |
| Source IP | `127.0.0.1` |
| Failure Reason | Invalid credentials |
| Timestamp | `[YOUR TIMESTAMP]` |
| Hostname | `[YOUR HOSTNAME]` |

---

## Analysis

The Windows Security log recorded an **Event ID 4625**, indicating that an authentication attempt failed.

The failed authentication was intentionally generated as part of this lab using an incorrect password.

Because the activity was generated locally against a test account, this specific event is **expected lab activity**.

---

## Security Relevance

Event ID 4625 becomes more interesting when multiple failures occur within a short period, especially when:

- The same account is repeatedly targeted.
- The source address is unexpected.
- Multiple accounts are targeted.
- Attempts occur at unusual times.
- A successful 4624 event follows the failures.

A pattern such as:

```text
4625
4625
4625
4625
    ↓
4624
```

could indicate a possible password-guessing or brute-force attempt and should be investigated further.

---

## Verdict

**Benign / Expected Lab Activity**

The event was intentionally generated for testing and does not indicate an actual attack.

---

## Recommended SOC Investigation

If this were a real environment, I would investigate:

1. Source IP reputation and ownership
2. Target account
3. Number of failed attempts
4. Logon type
5. Authentication method
6. Successful logons following failures
7. Activity after successful authentication
8. Related events from the same source

---

## Skills Learned

- Windows Security Logs
- Event ID 4625
- Authentication analysis
- Failed login investigation
- Brute-force detection concepts
- Event correlation
- SOC triage
- Evidence documentation

---

## Analyst Takeaway

A failed login is not automatically malicious.

The analyst must determine whether the event is:

```text
Normal
   ↓
Suspicious
   ↓
Malicious
```

using **context, frequency, source, timing, and event correlation**.