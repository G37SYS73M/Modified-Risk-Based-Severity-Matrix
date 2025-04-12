
# Modified Risk-Based Severity Matrix for Red Team Engagements

To appropriately assess the severity of findings discovered during the Red Team engagement, we utilized a **Modified Risk-Based Matrix** tailored specifically for offensive security operations. Unlike traditional vulnerability scoring systems (e.g., CVSS), this model incorporates not just the technical impact of a vulnerability or action, but also its **business relevance**, **real-world exploitability**, and **detectability** by defensive mechanisms.

This scoring system provides a more accurate representation of the **operational risk** posed by Red Team activities, enabling stakeholders to prioritize remediation efforts based on **real-world attack scenarios**.

---

## 📊 Scoring Dimensions

The Modified Risk-Based Matrix evaluates severity using three primary factors:

### 1. Impact

This metric represents the potential **business and technical consequence** of the attack or exploit.

| Impact Level | Description |
|--------------|-------------|
| **Critical** | Full compromise of crown jewels (e.g., domain controllers, production databases, sensitive customer data), persistent backdoor access, or exfiltration of confidential information. |
| **High**     | Compromise of critical infrastructure, lateral movement into restricted network segments, or access to regulated data (e.g., PII, PHI, PCI). |
| **Medium**   | User privilege escalation, access to sensitive but non-critical internal systems or data. |
| **Low**      | Initial access with limited privileges, successful phishing without internal movement, or discovery of misconfigurations with low business impact. |

### 2. Detectability

This dimension captures how effectively the activity was observed or responded to by the organization’s **defensive capabilities** (e.g., SOC, EDR, SIEM).

| Detection Level | Description |
|------------------|-------------|
| **Undetected**   | No alerts triggered, no telemetry collected, no response initiated. |
| **Partially Detected** | Some activity was logged or observed but not appropriately escalated or contained. |
| **Detected**     | Activity was identified and properly acted upon by defensive teams. |

### 3. Ease of Exploitation

This reflects the **level of effort, tooling, and skill** required to achieve the action or compromise.

| Ease Level | Description |
|------------|-------------|
| **Easy**   | Use of commodity tools or known exploits requiring minimal effort. |
| **Moderate**| Attack required chaining of multiple weaknesses or minimal customization. |
| **Hard**   | Required advanced techniques, custom tooling, or significant planning. |

---

### Risk Scoring Matrix

#### Qualitative Matrix

| Detectability \\ Impact | Critical | High | Medium | Low |
| ---------------------- | -------- | ---- | ------ | --- |
| **Undetected**         | Critical | Critical | High | Medium |
| **Partially Detected** | Critical | High     | Medium | Low |
| **Detected**           | High     | Medium   | Low    | Low |

This matrix is used to guide risk classification for Red Team findings, helping prioritize responses based on real operational scenarios, not just theoretical vulnerability scores.

---
## 🎯 Severity Scoring Logic

Each finding is assigned a **composite severity score** by evaluating the **combined effect of impact, detectability, and ease of exploitation**. This produces a score on a **scale from 0.0 to 10.0**, enabling prioritization similar to other frameworks, but grounded in the reality of adversarial simulation.

| Example Scenario | Impact  | Detection          | Ease     | Score |
|------------------|---------|--------------------|----------|-------|
| Domain Admin via Kerberoasting (undetected) | Critical | Undetected | Moderate | 9.5   |
| Phishing leading to workstation access (detected) | Medium   | Detected   | Easy     | 5.0   |
| Privilege escalation on DMZ host (no response) | High     | Partially Detected | Moderate | 7.0   |
| Misconfigured login portal with password spray | Low      | Detected   | Easy     | 2.5   |

---

## 🧠 Purpose and Benefits

This severity model provides:

- **Real-world risk prioritization** based on operational outcomes, not just static vulnerabilities.
- Enhanced value for **Blue Teams** by showing where gaps in detection exist.
- Flexibility to align with **business priorities**, including crown jewel protection, regulatory exposure, and response effectiveness.


