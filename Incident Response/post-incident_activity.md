# Incident Response – Post-Incident Activity

## Introduction

Every security incident provides valuable intelligence that can be used to strengthen an organization's security posture. The attacker’s methods, exploited vulnerabilities, failed controls, successful defenses, and response actions all provide lessons that can improve future detection and response capabilities.

According to NIST SP 800-61 Rev.2, the Post-Incident Activity phase focuses on identifying opportunities to improve the incident-handling process and enhance organizational preparedness. This phase is often overlooked, but it is where the long-term value of incident response is realized.

Organizations that skip this phase risk repeating the same mistakes and remaining vulnerable to similar attacks in the future.

---

## Lessons Learned Meeting

After an incident has been fully contained, eradicated, and recovered from, a Lessons Learned Meeting should be conducted.

This meeting typically includes:

- Incident Response Team
- SOC Analysts
- IT Administrators
- Security Engineers
- Management Stakeholders

The purpose of the meeting is to review the incident from start to finish and identify improvements.

Key questions addressed include:

- What happened?
- What was the root cause?
- How did the attacker gain access?
- How was the incident detected?
- Could the attack have been detected sooner?
- What worked well during the response?
- What could be improved?
- What changes should be implemented to prevent recurrence?

The answers gathered during this process are used to improve:

- Detection capabilities
- Security controls
- Incident response procedures
- Employee awareness
- Technology configurations

---

## Why Post-Incident Activity Matters

Post-Incident Activity is commonly skipped because:

- Response teams are exhausted after resolving the incident.
- Business stakeholders want operations restored quickly.
- Security teams must immediately focus on new alerts and threats.

However, failing to conduct a proper review means valuable intelligence is lost.

Every incident should contribute to improving:

- Detection engineering
- Security monitoring
- Incident response playbooks
- Organizational preparedness

---

## Closing the NIST Incident Response Cycle

The NIST Incident Response Lifecycle is a continuous improvement process.

Lessons learned during Post-Incident Activity are fed back into the Preparation phase.

This allows organizations to:

- Improve response procedures
- Develop new detection rules
- Enhance monitoring capabilities
- Strengthen security controls
- Train personnel more effectively

This cycle ensures that every incident contributes to a stronger security posture.

---

## Executive Summary

An Executive Summary is created for non-technical stakeholders such as:

- Chief Executive Officer (CEO)
- Chief Information Officer (CIO)
- Legal Teams
- Board Members
- Business Executives

The goal is to communicate the business impact of the incident without technical jargon.

A good executive summary should include:

| Category | Description |
|-----------|------------|
| What Happened | Overview of the attack and affected systems |
| Business Impact | Data loss, operational disruption, compliance concerns |
| How It Was Discovered & Resolved | High-level timeline of detection and response |
| Prevention & Remediation | Actions being taken to prevent recurrence |

---

## Technical Summary

The Technical Summary is written for technical teams including:

- SOC Analysts
- Incident Responders
- Security Engineers
- IT Administrators

The goal is to provide a complete record of the incident for future reference and security improvement.

A technical summary should contain:

| Category | Description |
|-----------|------------|
| Full Attack Timeline | Chronological sequence of attacker actions |
| Indicators of Compromise (IOCs) | Domains, IPs, accounts, files, and email addresses |
| MITRE ATT&CK Techniques | TTPs observed during the attack |
| Log Evidence | Queries and logs supporting findings |
| Root Causes | Security gaps that enabled the attack |
| Detection Gaps | Activities that were not detected initially |

---

## From TTPs to Detection Rules

Every Tactic, Technique, and Procedure (TTP) identified during an investigation can be used to improve detection capabilities.

By reviewing attacker behavior, analysts can create new detection rules that alert on similar activity in the future.

Examples include:

- Suspicious mailbox rule creation
- Unusual SharePoint file downloads
- Internal phishing activity
- OAuth application abuse
- Risky authentication attempts

This process creates a feedback loop where every incident improves future detection coverage.

---

## Detection as a Continuous Learning Process

No organization has perfect detection coverage.

Each incident reveals:

- New attacker techniques
- Visibility gaps
- Monitoring weaknesses

As incidents are investigated and new detections are created, the organization's security maturity increases over time.

The goal is to reduce:

- Detection time
- Investigation time
- Response time

for future incidents.

---

## Incident Response Tools

### Hawk

Hawk is an open-source PowerShell tool used to collect forensic data from Microsoft 365 environments.

Capabilities include:

- Mailbox analysis
- Sign-in investigations
- Administrative activity reviews
- Security log collection

---

### Sparrow

Sparrow was developed to identify Indicators of Compromise (IOCs) within Microsoft 365 and Azure environments.

Capabilities include:

- IOC hunting
- Azure investigations
- Microsoft 365 compromise assessments

---

### Microsoft Secure Score

Microsoft Secure Score measures an organization's overall security posture within Microsoft 365.

It helps organizations:

- Identify security gaps
- Prioritize remediation efforts
- Track security improvements
- Reduce attack surface

---

# Task 5 – Post-Incident Activity Practical

# Task 7 – Post-Incident Activity Practical

## Question 1

### Question

```text
What was the initial attack vector used to compromise Laura Chen's account?
```

### Investigation

During the post-incident review, the attack timeline was reconstructed to identify the root cause of the compromise. Email evidence, user activity, and authentication logs showed that Laura Chen received a malicious email containing a phishing link. The phishing page was designed to imitate a legitimate login portal and successfully captured the victim's credentials.

### Answer

```text
Phishing
```

### Evidence

---

## Question 2

### Question

```text
Even after the victim entered their credentials on the phishing page, what security control would have prevented the attacker from gaining access? (Please input the abbreviation of the term)
```

### Investigation

The Lessons Learned review identified that the attacker successfully authenticated using stolen credentials because an additional authentication factor was not required. Multi-Factor Authentication (MFA) would have introduced an extra verification step, preventing unauthorized access even if the password had been compromised.

### Answer

```text
mfa
```

### Evidence

---

## Question 3

### Question

```text
How many employees were put at risk by the internal phishing email?
```

### Investigation

After gaining access to the compromised mailbox, the attacker launched an internal phishing campaign. Message trace logs were reviewed to determine how many employees received the malicious email and were therefore exposed to the phishing attempt.

### Answer

```text
3
```

---

## Question 4

### Question

```text
What was the first log source that identified the suspicious activity?
```

### Investigation

The attack timeline was reviewed to determine where the first signs of malicious activity were detected. Authentication records within Microsoft Entra ID provided the earliest indication of suspicious sign-in activity associated with the compromised account.

### Answer

```text
Entra ID Sign-in Logs
```

### Evidence

---

## Question 5

### Question

```text
Which file downloaded by the attacker contains personally identifiable information of Nexus Financial employees?
```

### Investigation

SharePoint audit logs revealed several files accessed and downloaded by the attacker. During the review, one document was identified as containing sensitive employee information, including personally identifiable information (PII), representing a significant data exposure risk.

### Answer

```text
Full_Employee_PII_Data.xlsx
```

---

## Question 6

### Question

```text
For improving future detections, which Operation is relevant to highlight for detecting suspicious Inbox Rule Creations?
```

### Investigation

One of the attacker's persistence mechanisms involved creating malicious inbox rules to hide security notifications. During the Lessons Learned phase, analysts identified the mailbox operation associated with inbox rule creation that should be monitored and alerted on in future investigations.

### Answer

```text
New-InboxRule
```

---

## Question 7

### Question

```text
For improving future detections, which field in Entra ID logs can be used to detect authentications coming from unusual countries?
```

### Investigation

Reviewing sign-in logs showed that geographic information can be used to identify anomalous login activity. The relevant field in Entra ID logs provides country and region information, allowing analysts to create alerts for unexpected authentication locations.

### Answer

```text
location.countryOrRegion
```

---

# Lessons Learned

The incident highlighted several important security gaps and opportunities for improvement:

* Phishing remains one of the most effective initial access techniques.
* MFA could have prevented the compromise even after credential theft.
* Internal phishing campaigns can rapidly expand the scope of an incident.
* Early detection through authentication monitoring is critical.
* Sensitive files containing PII require additional monitoring and protection.
* Monitoring mailbox operations such as `New-InboxRule` can help detect persistence mechanisms.
* Geographic anomaly detection using `location.countryOrRegion` can improve visibility into suspicious authentication activity.

---

# Conclusion

The Post-Incident Activity phase transformed the incident into a valuable learning opportunity. By identifying the root cause, reviewing attacker behavior, documenting detection gaps, and recommending new monitoring controls, Nexus Financial strengthened its ability to detect and respond to similar attacks in the future. These lessons feed directly back into the Preparation phase of the NIST Incident Response Lifecycle, supporting continuous security improvement.

# Key Takeaways

- Every incident provides valuable intelligence.
- Lessons Learned meetings improve future response efforts.
- Executive and Technical Summaries serve different audiences.
- Detection engineering is a continuous process.
- MITRE ATT&CK mapping helps improve visibility and response.
- Security posture assessments should be conducted after every major incident.

---

# Conclusion

The Post-Incident Activity phase transforms an incident from a security failure into a learning opportunity. Through lessons learned meetings, technical reviews, detection engineering, and security posture improvements, organizations can strengthen their defenses and reduce the likelihood of future compromise. The insights gained during this phase feed directly back into the Preparation phase, completing the NIST Incident Response lifecycle and supporting continuous security improvement.
