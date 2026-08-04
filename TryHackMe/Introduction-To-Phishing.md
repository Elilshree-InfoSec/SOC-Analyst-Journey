# 🎣 Introduction to Phishing (SOC Simulator)

## 🎯 Objective

This scenario introduces the SOC Simulator and provides hands-on experience investigating phishing-related alerts. The goal is to practice the complete SOC L1 workflow, including alert triage, log investigation, report writing, prioritization, and escalation decisions.

---

## 📝 Summary

In this scenario, I acted as a SOC L1 Analyst responsible for investigating multiple phishing alerts. I reviewed alert details, searched for supporting evidence in Splunk SIEM, determined whether each alert was a True Positive or False Positive, documented my findings using the 5Ws, and decided whether escalation to L2 was required.

Unlike the previous theory rooms, this scenario focused on applying SOC workflows in a simulated environment.

---

## ⭐ Key Takeaways

- Practiced investigating alerts using Splunk SIEM.
- Learned to correlate alert information with log evidence.
- Applied the 5Ws when writing investigation reports.
- Classified alerts as True Positive or False Positive.
- Decided whether alerts required escalation.
- Applied alert prioritization based on severity before age.

---

# 🛠 SOC L1 Investigation Workflow

```text
New Alert
      ↓
Assign to Myself
      ↓
Set Status → In Progress
      ↓
Review Alert Details
      ↓
Search Logs in Splunk
      ↓
Collect Evidence
      ↓
Determine Verdict
      ↓
Write Investigation Report
      ↓
Decide on Escalation
      ↓
Close Alert
```

---

# 🔍 Investigation Process

For each alert, I followed these steps:

1. Assigned the alert to myself.
2. Reviewed the alert title, severity, and description.
3. Opened Splunk SIEM to search for related logs.
4. Compared the alert with available log evidence.
5. Determined whether the activity was malicious.
6. Wrote a report using the **5Ws**.
7. Decided whether escalation was necessary.
8. Closed the alert.

---

# 📄 Report Writing

For every investigation, I documented:

| Field | Purpose |
| :--- | :--- |
| **Who** | User or account involved |
| **What** | Suspicious activity detected |
| **When** | Time of the activity |
| **Where** | Host, IP address, or affected system |
| **Why** | Evidence supporting the final verdict |

---

# 🚦 Alert Prioritization

During the simulation, new alerts continued to appear while I was investigating existing ones.

I followed the prioritization workflow learned in previous rooms:

- Ignore alerts already assigned.
- Prioritize **Moderate** severity before **Low** severity.
- If alerts have the same severity, investigate the oldest one first.

This reinforced that **severity takes priority over age**.

---

# 💡 What I Practiced

- Alert triage
- Splunk log investigation
- Evidence collection
- Writing professional SOC reports
- Alert prioritization
- Escalation decision making
- Case management workflow

---

# 🧠 Skills Reinforced

- Splunk SIEM
- SOC Alert Triage
- Investigation Workflow
- Security Log Analysis
- Report Writing
- Critical Thinking
- Decision Making

---

# 💭 Personal Reflection

This was my first hands-on SOC simulation and helped me understand how a SOC L1 Analyst works in a real environment. Instead of only learning theory, I investigated live alerts, searched for supporting evidence in Splunk, and documented my findings like an analyst. One important lesson was applying proper alert prioritization—when a new Moderate severity alert appeared while an older Low severity alert was waiting, I investigated the Moderate alert first before returning to the older Low alert, following standard SOC procedures.

---

# 📅 Progress

- Scenario Completed: ✅
- Difficulty: Easy
