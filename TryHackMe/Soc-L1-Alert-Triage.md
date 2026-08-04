# 🚨 SOC L1 Alert Triage

## 🎯 Objective

This room introduces the fundamentals of SOC alerts and the alert triage process. It explains how alerts are generated, what information they contain, how analysts prioritize them, and the responsibilities of a Tier 1 SOC Analyst during an investigation.

---

## 📝 Summary

SOC analysts cannot manually review millions of security logs every day. Instead, security tools generate alerts whenever suspicious activity matches predefined detection rules. As a SOC L1 Analyst, the goal is to investigate these alerts, determine whether they represent a real threat, document findings, and escalate confirmed incidents when necessary.

---

## ⭐ Key Takeaways

- Alerts help analysts focus on suspicious activity instead of raw logs.
- Every alert contains important information for an investigation.
- L1 analysts perform the first investigation before escalating to L2.
- Alerts should be prioritized based on severity, status, and age.
- Investigation should always be evidence-based before reaching a verdict.

---

## 🔄 How Alerts Are Created

```text
Security Event
      ↓
System Log
      ↓
SIEM / EDR Collects Logs
      ↓
Detection Rule Matches
      ↓
Alert Generated
      ↓
SOC Analyst Investigates
```

The alert is simply a notification that tells analysts something unusual has happened. It does **not** automatically mean an attack has occurred.

---

## 🛠 Common Alert Management Platforms

| Platform | Purpose |
| :--- | :--- |
| **SIEM** | Collects logs and generates alerts. |
| **EDR** | Detects suspicious activity on endpoints. |
| **SOAR** | Automates repetitive investigation and response tasks. |
| **ITSM** | Tracks investigations and incident tickets. |

---

## 📋 Important Alert Fields

| Field | Why It Matters |
| :--- | :--- |
| **Alert Name** | Quick summary of the suspicious activity. |
| **Severity** | Indicates how urgent the alert is. |
| **Status** | Shows whether someone is already investigating it. |
| **Verdict** | Final classification after investigation. |
| **Assignee** | Analyst responsible for handling the alert. |
| **Description** | Explains why the alert was triggered. |
| **Alert Fields** | Contains useful investigation details such as usernames, hostnames, IPs, or commands. |

---

## 🚦 How Alerts Are Prioritized

Before investigating, analysts usually:

1. Ignore alerts already assigned to someone else.
2. Investigate **Critical** alerts before lower severity alerts.
3. If severity is the same, investigate the oldest alert first.

This helps ensure serious attacks are handled as quickly as possible.

---

## 🔍 Basic Alert Triage Process

```text
Assign Alert
      ↓
Set Status → In Progress
      ↓
Review Alert Details
      ↓
Investigate Evidence
      ↓
Determine Verdict
      ↓
Document Findings
      ↓
Escalate (if needed)
      ↓
Close Alert
```

---

## 🧠 Investigation Checklist

Before closing an alert, ask yourself:

- Who or what is affected?
- What activity triggered the alert?
- Is there supporting evidence in the logs?
- Are there related events before or after this alert?
- Is this normal behaviour?
- Should this be escalated?

---

# 🚦 Alert Severity Levels

| Severity | Meaning |
| :--- | :--- |
| 🟢 Low | Informational |
| 🟡 Medium | Moderate Risk |
| 🟠 High | Serious Threat |
| 🔴 Critical | Immediate Investigation Required |

---

## ⚖️ Possible Verdicts

| Verdict | Meaning |
| :--- | :--- |
| **True Positive** | The activity is malicious and requires action. |
| **False Positive** | The activity is legitimate and no threat exists. |

---

## 💡 Notes for Future Me

- An alert is **not** proof of an attack—it is a signal that needs investigation.
- Always investigate before making assumptions.
- Good documentation is part of the investigation.
- Prioritize work to avoid missing serious threats.
- Following a structured workflow reduces mistakes.

---

## 💭 Personal Reflection

This room helped me understand what a SOC L1 Analyst actually does on a daily basis. Instead of simply looking at alerts, analysts follow a structured process to investigate, classify, and document suspicious activity. I also learned that not every alert is malicious, making careful analysis and evidence gathering an essential part of the job.

---

## 📅 Progress

- Room Completed: ✅
- Difficulty: Easy
- Time Taken: 1 hour
