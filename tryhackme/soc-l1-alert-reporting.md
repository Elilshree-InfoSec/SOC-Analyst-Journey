# 📝 SOC L1 Reporting & Escalation

## 🎯 Objective

This room explains what happens after alert triage. It introduces alert reporting, escalation, and communication, helping SOC L1 analysts understand when to escalate incidents, how to write professional alert reports, and how to communicate effectively with teammates and other departments.

---

## 📝 Summary

Not every alert ends after triage. Once an investigation is complete, a SOC L1 analyst must document their findings, determine whether escalation is necessary, and communicate with the appropriate people. Good reporting helps senior analysts understand the investigation quickly, while proper escalation ensures serious threats receive the attention they require.

---

## ⭐ Key Takeaways

- Every investigation should be documented clearly.
- Reports provide context for future investigations.
- Escalate alerts that require deeper analysis or remediation.
- Communication is an important part of incident response.
- When unsure, always seek help instead of guessing.

---

# 🚨 Alert Lifecycle

```text
Alert Generated
      ↓
L1 Alert Triage
      ↓
Write Investigation Report
      ↓
Determine Verdict
      ↓
Close Alert
      OR
Escalate to L2
      ↓
L2 Investigation
      ↓
Incident Response (if required)
```

---

# 📊 The Alert Funnel

Most alerts never become security incidents.

```text
100 Alerts
     ↓
~90 False Positives / Resolved by L1
     ↓
~10 Escalated to L2
     ↓
~1 Major Incident (DFIR)
```

This highlights why L1 analysts play an important role in filtering and validating alerts before passing them to senior analysts.

---

# 📄 Why Alert Reports Matter

| Purpose | Why It Matters |
| :--- | :--- |
| **Provide Context** | Helps L2 understand the investigation without starting from scratch. |
| **Maintain Records** | Investigation notes remain available even after logs expire. |
| **Improve Skills** | Writing reports reinforces analytical thinking and investigation skills. |

---

# ✍️ The 5W Reporting Method

A good SOC report should answer these five questions:

| Question | Include |
| :--- | :--- |
| **Who** | User, account, or process involved |
| **What** | Suspicious activity detected |
| **When** | Date and time of the activity |
| **Where** | Hostname, IP address, website, or affected system |
| **Why** | Reason for your verdict and supporting evidence |

---

# 🚩 When Should You Escalate?

Escalate the alert if:

- The activity indicates a serious cyberattack.
- Additional investigation is required.
- Remediation actions are needed.
- Multiple departments must be involved.
- You are unsure how to classify the alert.

**Remember:** It is better to ask for help than incorrectly close a malicious alert.

---

# 🔄 Escalation Process

```text
Complete Investigation
      ↓
Write Alert Report
      ↓
Set Verdict
      ↓
Assign Alert to L2
      ↓
Notify L2
      ↓
L2 Continues Investigation
```

---

# 🤝 Communication During Investigations

SOC analysts may need to communicate with:

| Team | Purpose |
| :--- | :--- |
| **IT Team** | Verify system changes or user permissions |
| **HR** | Confirm employee information |
| **L2 Analysts** | Request guidance or escalate incidents |
| **SOC Manager** | Report major issues or urgent situations |

---

# ⚠️ Common Communication Scenarios

| Situation | Recommended Action |
| :--- | :--- |
| L2 is unavailable | Contact L2, then L3, then your manager. |
| User's Teams/Slack account is compromised | Use another communication method such as a phone call. |
| Large number of incoming alerts | Prioritize alerts and notify L2. |
| You realize you made the wrong verdict | Inform L2 immediately. |
| SIEM logs are unavailable | Investigate what you can and report the issue. |

---

# 💡 Best Practices

- Write clear and concise investigation reports.
- Support every verdict with evidence.
- Never assume an alert is malicious or benign.
- Escalate whenever additional expertise is required.
- Communicate using approved channels.
- Keep investigation notes professional and objective.

---

# 🎯 Interview Notes

A SOC L1 Analyst is responsible for more than just closing alerts.

Their responsibilities include:

- Investigating alerts
- Writing investigation reports
- Classifying alerts
- Escalating when necessary
- Communicating with internal teams
- Maintaining accurate documentation

One of the most useful reporting techniques is the **5Ws**:

- **Who**
- **What**
- **When**
- **Where**
- **Why**

---

# 💭 Personal Reflection

This room helped me understand that alert triage is only one part of a SOC analyst's job. Good documentation, clear communication, and knowing when to escalate are equally important. I also learned that asking for help is encouraged when handling unfamiliar alerts, as accurate investigations are more important than making assumptions.

---

# 📅 Progress

- Room Completed: ✅
- Difficulty: Easy
- Time Taken: 1 hour
