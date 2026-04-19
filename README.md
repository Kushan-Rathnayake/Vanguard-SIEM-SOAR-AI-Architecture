# 🛡️ Vanguard: SIEM, SOAR, & AI-SOC Architecture
A vendor-agnostic, closed-loop SOC &amp; SOAR architecture. Features adversary emulation (Atomic Red Team), SIEM detection engineering (Wazuh/Sysmon), serverless automation (Tines), and AI-driven incident triage (Groq/Llama-3) with a Human-in-the-Loop (HITL) active response workflow.

<p align="center">
  <img src="Assets/19.) Architectural Diagram.png" alt="Project Vanguard Architecture Diagram">
  <br>
  <em>Figure 1: High-level closed-loop architecture mapping the full lifecycle from emulation to AI-triaged HITL remediation.</em>
</p>

## 🎯 Project Objective
**Project Vanguard** is a vendor-agnostic, open-source Security Operations Center (SOC) home lab. The objective of this project was to move beyond theory and build a functional, closed-loop detection and response pipeline. By combining adversary emulation, custom detection engineering, serverless automation, and LLM-driven triage, this project demonstrates how to ingest raw endpoint telemetry and transform it into actionable, enriched intelligence for human analysts.

## 🛠️ Infrastructure & Tech Stack
* **Cloud IaaS:** Microsoft Azure (Ubuntu Server 22.04 LTS / `Standard_D4s_v3`)
* **Endpoint & Telemetry:** Windows 11, Sysmon (SwiftOnSecurity Config)
* **Adversary Emulation (BAS):** Atomic Red Team (ART)
* **SIEM / XDR:** Wazuh 4.x
* **SOAR (Automation):** Tines (Serverless Enterprise Pipeline)
* **AI & Threat Intel:** Groq API (Llama-3.1-8b-instant LLM)
* **Case Management & ChatOps:** Jira Cloud, Slack

---

## 🏗️ Phase 1: Infrastructure & Hardening
To replicate a production environment without inflating cloud costs, the entire backend runs on a single hardened Azure VM. 

Instead of leaving the cloud instance open to the internet, I configured strict Network Security Group (NSG) rules. The environment only permits inbound traffic on explicitly required ports: Port 22 (SSH - Restricted to Admin IP), Port 443 (Wazuh UI), Ports 1514/1515 (Agent Communication), and Port 55000 (Wazuh API).

<details>
<summary>📸 View Infrastructure Setup Proofs</summary>
<br>
<img src="Assets/Setup/1.) Vanguard-VM (Resource Overview).png" width="800">
<img src="Assets/Setup/2.) Network Security Group (NSG) Rules.png" width="800">
<img src="Assets/Setup/3.) WindowsTerminal_99kguNCOMG.png" width="800">
</details>

---

## 📡 Phase 2: Endpoint Telemetry & Log Ingestion
Visibility is the foundation of any SOC. I deployed a local Windows 11 endpoint and installed the Wazuh Agent alongside **Sysmon**. By applying the SwiftOnSecurity Sysmon configuration, the endpoint captures granular telemetry (Process Creation, Network Connections, etc.) and streams it securely to the Azure-hosted Wazuh Manager.

<details>
<summary>📸 View Telemetry Pipeline Proofs</summary>
<br>
<img src="Assets/Setup/6.) Sysmon Installation.png" width="800">
<img src="Assets/Setup/4.) Wazuh Dashboard (Telemetry-Pipeline-Active).png" width="800">
</details>

---

## ⚔️ Phase 3: Adversary Emulation & Detection Engineering
To validate the SIEM pipeline, I utilized **Atomic Red Team** to emulate **MITRE T1059.001 (PowerShell)**. I executed a simulated malicious Download Cradle via the command line.

Instead of relying on default alerts, I engineered a **Custom Level 15 Critical Wazuh Rule**. This rule uses regex to parse Sysmon Event ID 1 (Process Creation) and specifically map malicious command-line arguments (like `-ExecutionPolicy Bypass`) to their respective MITRE ATT&CK IDs. 

<details>
<summary>📸 View Attack & Custom Detection Proofs</summary>
<br>
<img src="Assets/Setup/15.) Atomic Red Team Execution (1).png" width="800">
<img src="Assets/Setup/8.) Raw-Sysmon-Telemetry.png" width="800">
<img src="Assets/Setup/10.) Custom Detection XML.png" width="800">
<img src="Assets/Setup/9.) Custom Detection Level-15.png" width="800">
</details>

---

## 🧠 Phase 4: SOAR Integration & Alert Aggregation
Generating alerts is easy; preventing alert fatigue is the real challenge. I configured Wazuh's `ossec.conf` to push custom JSON webhooks to **Tines** (SOAR platform). 

**The Aggregation Engine:** To combat alert storms caused by multi-stage attacks, I engineered a 90-second data batching window in Tines. This correlates incoming events by endpoint hostname, waiting for the attack chain to finish before compiling the telemetry into a single, clean JSON array for analysis.

<p align="center">
  <img src="Assets/12.) Tines SOAR Architecture Canvas.png" width="900" alt="Tines SOAR Pipeline">
</p>

<details>
<summary>📸 View Data Correlation Proof</summary>
<br>
<img src="Assets/Setup/13.) Data Correlation Engine (aggregate_alerts_by_hostname).png" width="800">
</details>

---

## 🤖 Phase 5: AI-Driven Analysis & Ticketing
Once the aggregated telemetry is batched, Tines triggers a webhook to the **Groq API**, feeding the raw JSON to the **Llama-3.1-8b-instant** model. 

Using strict system prompt engineering, the LLM acts as a Tier 1 SOC Analyst. It parses the base64-encoded strings and command-line arguments, evaluates the threat, and formats a structured summary. Tines then uses this AI output to generate a highly detailed, color-coded incident ticket in **Jira Cloud** via Wiki Markup.

<p align="center">
  <img src="Assets/17.) Automated Jira Incident Ticket (Full Image).png" width="900" alt="Jira Incident Ticket">
</p>

<details>
<summary>📸 View Prompt Engineering Proof</summary>
<br>
<img src="Assets/Setup/14.) Groq LLM System Prompt (executive_ai_analyst).png" width="800">
</details>

---

## 🚨 Phase 6: Human-in-the-Loop (HITL) Active Response
A critical engineering decision in this architecture was avoiding a "blind" automated kill-switch. Unauthenticated API isolation requests frequently cause self-inflicted network outages (False Positives).

Instead, I built a **Human-in-the-Loop (HITL)** workflow. The SOAR pipeline pushes an interactive page to the SOC Slack channel. The "Isolate Host" button provides a secure, **authenticated deep-link** that drops the analyst directly into the Wazuh Dashboard for that specific endpoint. The analyst must log in, review the AI triage, and manually execute the isolation command.

<p align="center">
  <img src="Assets/18.) Slack HITL Active Response (GIF).gif" width="800" alt="Slack Active Response Workflow">
</p>

---

## 🗺️ Phase 7: Threat Coverage & Gap Analysis
Security is an ongoing process of visibility mapping. Following the MITRE ATT&CK framework, I mapped the outcomes of the Atomic Red Team emulations against my pipeline to validate defenses and identify blind spots.

<p align="center">
  <img src="Assets/20.) ATT&CK Mapping 1.png" width="900" alt="MITRE ATT&CK Heatmap">
  <br>
  <em>Figure 2: Custom MITRE Navigator Heatmap.</em>
</p>

* 🟢 **Mitigated (Green):** Telemetry caught, analyzed by SOAR/AI, and successfully routed to Slack for HITL isolation (e.g., T1059.001 - PowerShell, T1105 - Ingress Tool Transfer).
* 🟡 **Detected (Yellow):** Telemetry ingested and flagged by Wazuh, but currently requires heavy tuning to prevent alert fatigue from legitimate admin activity (e.g., T1027 - Obfuscated Files).
* 🔴 **Visibility Gap (Red):** Emulated HTTPS C2 traffic successfully bypassed detection due to a lack of SSL/TLS decryption. Integrating a Network IDS (like Suricata) is the next immediate objective for this lab.
