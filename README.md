# Modified Risk-Based Severity Matrix Calculator

A web app implementing a red-team–focused severity scoring model that combines **Impact**, **Likelihood**, and **Detectability** into a single composite Risk Score. This approach extends traditional 5×5 risk matrices by treating “Detectability” (how easily defenders will notice an attack) as a third dimension, yielding a 1–125 score and mapping it to actionable tiers (Low/Medium/High/Critical).

---

## Table of Contents

- [Introduction](#introduction)  
- [Background & Motivation](#background--motivation)  
- [Model Overview](#model-overview)  
  - [Impact](#impact)  
  - [Likelihood (Exploitability)](#likelihood-exploitability)  
  - [Detectability](#detectability)  
- [Composite Scoring & Risk Tiers](#composite-scoring--risk-tiers)  
- [Web Application](#web-application)  
  - [Live Demo](#live-demo)  
  - [How to Use](#how-to-use)  
- [Tech Stack](#tech-stack)  
- [Contributing](#contributing)  
- [License](#license)  

---

## Introduction

Traditional vulnerability scoring systems (e.g., CVSS) and 2-D risk matrices measure “Impact” and “Likelihood” but ignore an attacker’s ability to evade detection. In modern red-team engagements, whether an exploitation attempt is caught by monitoring is often the difference between a minor incident and a full-blown breach. This project implements the **Modified Risk-Based Severity Matrix** described in the accompanying white paper (see “Background & Motivation”).

Rather than returning a plain CVSS number or a simple color on a 5×5 grid, our model produces:

1. **Numeric Risk Score (1–125)**  
2. **Categorical Tier (Low/Medium/High/Critical)**  
3. **Tier-specific guidance** (e.g., response timeline)

By explicitly quantifying **Detectability**—how likely your existing defenses (IDS/IPS/SIEM, logs, alerting) would catch the attack—this model ensures “quiet” vulnerabilities (easy for an attacker to exploit unnoticed) receive appropriately high priority.

---

## Background & Motivation

1. **CVSS Limitations:**  
   - CVSS v3.x/4.0 provides a standardized technical severity rating (0.0–10.0) for software vulnerabilities, but it does not account for **organizational context** (e.g., business impact, compensating controls, or detection capabilities).  
   - CVSS measures “exploitability” factors (Attack Complexity, Privileges Required, etc.) in a vacuum. Two equally “Complex” exploits might be trivial in one environment and impossible in another.

2. **SSVC & Other Frameworks:**  
   - The Stakeholder-Specific Vulnerability Categorization (SSVC) framework improves context by involving business stakeholders, but its multi-step flowchart can be cumbersome and yields only categorical outputs (e.g., “Fix Now” vs. “Monitor”).  
   - Failure-Mode and Effects Analysis (FMEA) introduces a “Detection” dimension (Severity × Occurrence × Detection), but it was designed for engineering failures—not cyber risk.

3. **Why Detectability Matters:**  
   - In many red-team exercises, a “Critical” CVSS vulnerability is blocked or logged immediately. Meanwhile, a “Moderate” attack that slips past logging can remain undetected for weeks, causing greater harm.  
   - By explicitly scoring Detectability on a 1–5 scale (where higher means harder to detect), we capture **dwell-time risk**: the potential damage caused by an attacker operating unnoticed.

---

## Model Overview

The model scores each finding on **three 5-point scales**:

1. **Impact (I)**  
2. **Likelihood (L)**  
3. **Detectability (D)**  

The final **Composite Risk Score** is simply:
Risk Score = I × L × D (range: 1 … 125)

Each dimension uses the following rubric:

### Impact

Measures the potential damage to the organization if the finding is exploited (technical + business impact).  
1. **Negligible (1)** ● No significant harm or business effect  
2. **Minor (2)** ● Limited, localized impact (e.g., non-critical data exposure to a small group)  
3. **Moderate (3)** ● Noticeable impact (e.g., subset of sensitive records, short downtime)  
4. **Major (4)** ● Serious impact (e.g., large data breach, extended outage)  
5. **Critical (5)** ● Catastrophic (e.g., systemic failure, regulatory fines, > \$1 million loss)

> _Scorers should ask: “If an adversary succeeds here, how bad could it be for us (in dollar value or reputation)?”_

---

### Likelihood (Exploitability)

Gauges how probable it is that the vulnerability or scenario could be exploited, blending technical ease + attacker motivation + environmental context.  
1. **Rare (1)** ● Extremely unlikely or requires highly advanced capabilities  
2. **Unlikely (2)** ● Possible but not expected under normal threat conditions  
3. **Possible (3)** ● Feasible given known threats; moderate effort or some preconditions needed  
4. **Likely (4)** ● Straightforward for skilled attackers or already observed in the wild  
5. **Certain (5)** ● Exploitation is trivial or actively happening (e.g., public exploit code, mass scanners)

> _Unlike CVSS’s purely technical “Attack Complexity,” this scale also factors in how easy it is for a real-world adversary to reach and use the vulnerability in your environment._

---

### Detectability

Assesses how easily your existing defenses (logs, SIEM, IDS/IPS, EDR) will detect or alert on the exploitation. Higher = harder to detect (worse).  
1. **Very High Detectability (1)** ● Almost certain to see it immediately (noisy behavior, known signature)  
2. **High Detectability (2)** ● Likely caught by existing controls (automated monitoring, SIEM rules)  
3. **Moderate Detectability (3)** ● Even chance of detection; may slip by initially but found on investigation  
4. **Low Detectability (4)** ● Unlikely to be caught timely; attacker can blend in unless hunted  
5. **Very Low Detectability (5)** ● Essentially stealthy; little to no visibility, can dwell for a long time

> _I.e., “If an attacker tried this, would my SIEM/EDR/IDS see it?” A Detectability score of 5 means it’s virtually invisible._

---

## Composite Scoring & Risk Tiers

Once you pick **I**, **L**, and **D**, the app computes:
Risk Score = I × L × D (1 ≤ score ≤ 125)

We then map the numeric score into **four qualitative tiers**:

| Tier     | Score Range | Definition                                                                                       | Action                                                                                                 |
|----------|-------------|--------------------------------------------------------------------------------------------------|--------------------------------------------------------------------------------------------------------|
| **Low** (Sev 1)     | < 20        | Low risk – low impact and/or almost certain to be detected.                               | Accept or defer. Fix during routine maintenance; track for reevaluation if the environment changes.    |
| **Medium** (Sev 2)  | 20 – 49   | Moderate risk – either high impact but high detectability, or lower impact with low detectability. | Scheduled remediation within 30–60 days. Use compensating controls or heightened monitoring in the interim. |
| **High** (Sev 3)    | 50 – 74     | High risk – serious impact and/or low detectability; limited early warning.                   | Prompt remediation within the next patch cycle (e.g., 7 days). Apply temporary controls if needed.      |
| **Critical** (Sev 4)| ≥ 75        | Critical risk – extreme impact and very low detectability (stealthy).                           | Immediate mitigation. Emergency patching or incident response within 24–48 hours. Inform senior management. |

> _You can adjust these cut-offs to match your organization’s appetite (e.g., treat ≥ 90 as Critical), but the key is: every tier has a clear SLA._

---

## Web Application

A single-page app implementing the Modified Risk-Based Severity Matrix. It’s written in plain **HTML**, **CSS**, and **JavaScript** and runs entirely in your browser—no backend required.

### Live Demo
👉 [https://riskcalculator.adversarialops.com/](https://riskcalculator.adversarialops.com/)

---

### How to Use

1. **Open the Live App**  
   Navigate to [https://riskcalculator.adversarialops.com/](https://riskcalculator.adversarialops.com/) in any modern browser.

2. **Select Impact (1–5)**  
   - 1 – Negligible  
   - 2 – Minor  
   - 3 – Moderate  
   - 4 – Major  
   - 5 – Critical  

3. **Select Likelihood (1–5)**  
   - 1 – Rare  
   - 2 – Unlikely  
   - 3 – Possible  
   - 4 – Likely  
   - 5 – Certain  

4. **Select Detectability (1–5)**  
   - 1 – Very High (easy to detect)  
   - 2 – High  
   - 3 – Moderate  
   - 4 – Low  
   - 5 – Very Low (stealthy)  

5. **Click “Calculate Risk”**  
   The app instantly computes:

Risk Score = Impact × Likelihood × Detectability

and displays:  
- **Numeric Risk Score** (1–125)  
- **Severity Tier** (Low/Medium/High/Critical)  
- **Tier-specific guidance** (response timeline, recommended actions)

6. **Interpret & Act**  
Follow the guidance in the tier description:  
- **Critical** (≥ 75): Immediate mitigation, inform leadership.  
- **High** (50–74): Patch within 7 days, monitor.  
- **Medium** (20–49): Schedule remediation in 30–60 days.  
- **Low** (< 20): Defer to routine maintenance; track for review.

---

## Tech Stack

- **HTML5** – Markup for form, dropdowns, and output.  
- **CSS3** – Responsive, mobile-friendly styling; color-coded buttons and output box.  
- **Vanilla JavaScript** – Reads dropdown values, computes `I × L × D`, maps to the correct tier, and updates the DOM.

_No frameworks or external libraries are used—this is deliberately lightweight so it can run offline or be embedded in any red-team report._

---

## Contributing

1. **Clone this repo**  
```bash
git clone https://github.com/yourusername/Modified-Risk-Based-Severity-Matrix.git
cd Modified-Risk-Based-Severity-Matrix
