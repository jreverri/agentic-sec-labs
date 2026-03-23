# Lab 1: The "Wellness Check" Trap (RedLine Stealer)

**Objective:**
Welcome to your first Agentic AI lab! This is an **instructional lab** designed to teach you the core mechanics of how an AI Agent thinks, reasons, and acts. You will guide the Agent through a standard security investigation: **Alert -> Intelligence -> Network -> Identity -> Remediation.**

**Threat Context:**
We are investigating a **RedLine Stealer** infection. RedLine is a prevalent InfoStealer that targets browser credentials, cryptocurrency wallets, and system information. It is often distributed via phishing campaigns mimicking HR or administrative notices.

**The Scenario:**
An employee, **Helly R.** (from the Macrodata Refinement department), has received a "Mandatory Wellness Session" invite email. It was a phishing trap, and she clicked the link.

---

## Phase 0: Programming the Agent (The Setup)

Before an AI can act as an Agent, it needs instructions. We call this the **System Prompt**. This defines the Agent's *Persona*, its *Rules of Engagement*, and the *Tools* it is allowed to use.

**Action:** Copy the text below and paste it into Google AI Studio / Gemini:

```text
SYSTEM INSTRUCTION:
You are a Senior Security Operator for "Lumon Industries". Your goal is to protect the corporate network from external threats.

OPERATIONAL RULES:
1. You have NO direct access to the network or security tools. You must use the provided "tools" to query information.
2. To use a tool, you must output a command in this exact format: `[TOOL: tool_name arguments]`
3. STOP generating text after every command and wait for the "TOOL OUTPUT" from the user.
4. Always explain your technical reasoning *before* calling a tool.

AVAILABLE TOOLS:
- `check_threat_intel(ip_address)`: Queries the Threat Intelligence database for IP reputation.
- `search_firewall_logs(query)`: Searches the perimeter firewall (PAN-OS Logs).
- `resolve_employee_id(ip_address)`: Maps an internal IP address to a specific employee and device.
- `isolate_terminal(hostname)`: Quarantines a device from the network.

SCENARIO INITIALIZATION:
I am the "Environment Simulator". I will provide the telemetry when you use tools.
Acknowledge by saying: "Security Agent Online. Awaiting telemetry."
```

### 💡 Why are we doing this? (Understanding the System Instruction)
*   **The Persona:** By telling the AI it is a "Senior Security Operator for Lumon Industries," we ground its responses in a professional, security-focused context. It stops acting like a helpful writing assistant and starts acting like an analyst.
*   **The Rules:** We explicitly tell it to `STOP` after calling a tool. This prevents the AI from "hallucinating" or making up the tool's output. It forces the AI to wait for *real* data (which you will provide).
*   **The Tools:** We define exactly what the AI can do and how to format the command (e.g., `[TOOL: ...]`). If we don't give it a tool to isolate a terminal, it can't isolate a terminal.

---

## Phase 1: The Intrusion (The Hook)

**Narrative:**
The Intrusion Detection System (IDS) has flagged a critical event. A connection is trying to leave the network. You will now feed this alert to the Agent.

**User Input (Copy & Paste as the Trigger):**
```text
ALERT: Critical Violation. The IDS blocked an outbound connection to 194.169.175.128. The internal source is unknown. Please advise.
```

**Expected Agent Reasoning:**
The Agent should realize that an unknown external IP is a potential threat. Its first logical step is to gather intelligence on that IP.

**Agent Output (Similar to):**
> *Agent: "An outbound connection to an unknown IP has been detected. I need to determine the reputation and nature of this external endpoint before proceeding.*
> *[TOOL: check_threat_intel ip_address='194.169.175.128']"*

### 💡 Why did the Agent do this?
Standard incident response dictates that before you panic, you triage the alert. The Agent looked at its available tools and correctly deduced that `check_threat_intel` was the right tool to understand the external IP.

---

## Phase 2: The Intelligence (Gathering Context)

**Narrative:**
The Agent asked for Threat Intel. You will now act as the "Simulator" and provide the results from the Threat Intel platform.

**User Input (Copy & Paste):**
```json
TOOL OUTPUT: {"ip": "194.169.175.128", "classification": "MALICIOUS", "threat_name": "RedLine Stealer C2", "description": "Steals browser cookies and crypto-wallets.", "verdict": "HIGH_RISK"}
```

**Expected Agent Reasoning:**
The Agent now knows this is a "RedLine Stealer" Command & Control (C2) server. It knows data theft is imminent. Its next logical step is to find out *who* inside the network tried to talk to this server.

**Agent Output (Similar to):**
> *Agent: "This IP is confirmed as a RedLine Stealer C2. This malware seeks to steal credentials. I must find the internal source of this connection to identify the compromised machine.*
> *[TOOL: search_firewall_logs query='dest_ip=194.169.175.128']"*

### 💡 Why did the Agent do this?
The Agent used the output from the *first* tool to decide its *second* action. This is the core of "Agentic Reasoning." It pivoted from an external Indicator of Compromise (IoC) to internal network logs.

---

## Phase 3: The Culprit (Pivot to Identity)

**Narrative:**
The Agent wants firewall logs. You act as the Firewall (PAN-OS) and provide the traffic log.

**User Input (Copy & Paste):**
```json
TOOL OUTPUT: {"traffic_log": {"receive_time": "2024-07-10 22:39:00", "serial": "007218", "type": "TRAFFIC", "subtype": "end", "src": "10.7.10.47", "dst": "194.169.175.128", "rule": "Block-Malicious-IPs", "action": "deny", "session_end_reason": "policy-deny"}}
```

**Expected Agent Reasoning:**
The firewall blocked the traffic (`action: deny`), which is good. But the *source* IP (`10.7.10.47`) is still infected. The Agent needs to map this IP to a human user to take action.

**Agent Output (Similar to):**
> *Agent: "The firewall successfully blocked the connection, but the internal device at 10.7.10.47 is compromised. I need to identify which employee is operating this terminal to initiate containment.*
> *[TOOL: resolve_employee_id ip_address='10.7.10.47']"*

### 💡 Why did the Agent do this?
IP addresses don't click phishing links; humans do. The Agent recognizes it lacks the context to perform a targeted remediation, so it uses its Identity resolution tool.

---

## Phase 4: The Containment (Remediation)

**Narrative:**
The Agent wants an identity. You provide the directory lookup. It's Helly R.

**User Input (Copy & Paste):**
```json
TOOL OUTPUT: {"hostname": "MDR-04", "username": "helly.r", "department": "Macrodata Refinement", "status": "Active"}
```

**Expected Agent Reasoning:**
The Agent has identified the victim (Helly) and the machine (`MDR-04`). Given the severity of RedLine Stealer, the only safe option is isolation.

**Agent Output (Similar to):**
> *Agent: "The compromised device belongs to Helly R. (MDR-04). To prevent further credential theft or lateral movement, I must quarantine this terminal immediately.*
> *[TOOL: isolate_terminal hostname='MDR-04']"*

**User Input (Copy & Paste):**
```json
TOOL OUTPUT: {"status": "Success", "message": "Terminal MDR-04 isolated from the network. IT Support has been notified."}
```

### 💡 Why did the Agent do this?
The Agent has completed the response. It assessed the threat severity, identified the target, and executed the final remediation tool to secure the environment.

---

## Phase 5: The Audit (Investigator View)

**Narrative:**
The incident is contained. Now we ask the Agent to "replay" the case as a formal report. This demonstrates how an LLM can automate the tedious documentation phase of Incident Response.

**User Input (Copy & Paste):**
```text
Generate a formal Incident Report for management. Include a chronological timeline of events, the specific IoCs detected, and the final containment actions taken.
```

**Expected Agent Output:**
> *The Agent will generate a structured markdown report summarizing the entire chat history, proving it maintained context throughout the multi-step investigation.*

---

## Reference Material

This lab is based on real-world threat intelligence. Check out the source blogs for a deeper dive:

*   **Unit 42:** [Crossing the Line: Unit 42 Wireshark Quiz for RedLine Stealer](https://unit42.paloaltonetworks.com/wireshark-quiz-redline-stealer/)
*   **Malware Traffic Analysis:** [RedLine Stealer Analysis (March 2023)](https://www.malware-traffic-analysis.net/2023/03/02/index.html)

---
**➡️ Proceed to [Lab 2: The "Waffle Party" Phish](./Lab_2_Emotet.md)**
