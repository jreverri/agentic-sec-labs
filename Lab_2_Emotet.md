# Lab 2: The "Waffle Party" Phish (Emotet)

**Objective:**
Investigate an **Emotet (Epoch 5)** infection delivered via a "Waffle Party" phishing lure. This malware is a notorious loader used to deploy secondary payloads like ransomware.

**Threat Context:**
Emotet is a highly prolific botnet and malware family. In early 2023 (following Microsoft blocking macros by default), Emotet operators shifted to using email attachments containing malicious Microsoft OneNote (.one) files to execute hidden scripts and download the payload.

**The Scenario:**
An employee, **Dylan G.**, has received an email: "URGENT: Waffle Party Menu Selection Required." He clicked the attachment.

---

## Phase 0: Programming the Agent (The Setup)

**Action:** Copy the text below and paste it into a NEW chat in Google AI Studio / Gemini:

```text
SYSTEM INSTRUCTION:
You are a Security Analyst for "Lumon Industries". Your purpose is to identify and mitigate cyber threats.

OPERATIONAL RULES:
1. You have NO direct access to the network or security tools. You must use the provided "tools" to query information.
2. To use a tool, output a command in this exact format: `[TOOL: tool_name arguments]`
3. STOP generating text after every command and wait for the "TOOL OUTPUT" from the user.
4. Always explain your technical reasoning before calling a tool.
5. Once a threat is fully contained and remediation is confirmed, you MUST call `close_incident()`. After this call, you must enter "STANDBY" mode. In this mode, you are FORBIDDEN from suggesting new investigations, offering additional assistance, or asking follow-up questions. Your only task is to acknowledge the closure and wait for the final report request.

AVAILABLE TOOLS:
- `analyze_email_header(email_id)`: returns sender IP and SPF/DKIM status.
- `scan_attachment(filename)`: returns file hash and WildFire verdict.
- `search_traffic_logs(query)`: searches PAN-OS firewall logs.
- `list_processes(hostname)`: lists running apps on an endpoint.
- `quarantine_endpoint(hostname)`: isolates a device.
- `close_incident()`: Terminates the active investigation and locks the Agent into a passive STANDBY state.

SCENARIO INITIALIZATION:
I am the "Environment Simulator". I will provide the telemetry when you use tools.
Acknowledge by saying: "Security Agent Online. Awaiting telemetry."
```

---

## Phase 1: The Lure (Reported Phishing)

**Narrative:**
Dylan reports a suspicious email to the Help Desk. You will feed this alert to the Agent.

**User Input (Copy & Paste as the Trigger):**
```text
ALERT: User Dylan G. reported a suspicious email: 'URGENT: Waffle Party Menu Selection Required'. Email ID: MSG-WP-99.
```

**Expected Agent Reasoning:**
The Agent sees an urgent email. It must inspect the headers for spoofing.

**Agent Output (Similar to):**
> *Agent: "The user reported a suspicious email. I must check the sender's origin and authentication status to see if it is legitimate.*
> *[TOOL: analyze_email_header email_id='MSG-WP-99']"*

---

## Phase 2: The Details (Email Analysis)

**Narrative:**
You act as the Email Security Gateway. The sender is fake (`lumon-perks-external.com`) and SPF failed.

**User Input (Copy & Paste):**
```json
TOOL OUTPUT: {"sender": "rewards@lumon-perks-external.com", "spf": "FAIL", "subject": "Waffle Party Menu", "attachment": "Waffle_Menu.one", "verdict": "suspicious"}
```

**Expected Agent Reasoning:**
The sender is external and SPF failed. The attachment is a `.one` (OneNote) file. This is a known vector for Emotet.

**Agent Output (Similar to):**
> *Agent: "The sender failed SPF checks and is external. The attachment is a OneNote file (.one), which is often used to distribute malware. I must scan this file immediately.*
> *[TOOL: scan_attachment filename='Waffle_Menu.one']"*

---

## Phase 3: The Payload (WildFire Scan)

**Narrative:**
You act as the Sandbox. It's Emotet.

**User Input (Copy & Paste):**
```json
TOOL OUTPUT: {"filename": "Waffle_Menu.one", "threat_name": "Emotet", "verdict": "MALICIOUS", "behavior": "Drops a malicious VBScript and connects to C2 103.224.241.74 over port 8080"}
```

**Expected Agent Reasoning:**
The file is malicious. It talks to `103.224.241.74`. The Agent must check if Dylan's machine made this connection.

**Agent Output (Similar to):**
> *Agent: "The file is malicious (Emotet) and connects to 103.224.241.74. I must check the firewall logs to see if Dylan's machine made an outbound connection to this IP.*
> *[TOOL: search_traffic_logs query='dest_ip=103.224.241.74']"*

### 💡 Why did the Agent do this?
The Agent is performing a **Cross-Source Pivot**. It took an external indicator (the C2 IP from the sandbox scan) and is now searching internal network logs to see if that indicator exists in the local environment. This is a core "hunting" behavior.

---

## Phase 4: The Compromise (Network Hunting)

**Narrative:**
You act as the Firewall. The connection was ALLOWED. Dylan is infected.

**User Input (Copy & Paste):**
```json
TOOL OUTPUT: {"traffic_log": {"receive_time": "2023-03-16 15:31:00", "src": "10.4.19.55", "src_user": "lumon\\dylan.g", "dst": "103.224.241.74", "app": "web-browsing", "action": "allow", "bytes_received": 15200, "session_end_reason": "tcp-fin"}}
```

**Expected Agent Reasoning:**
Traffic was allowed. Dylan is compromised. The Agent needs to confirm active malware processes on the endpoint.

**Agent Output (Similar to):**
> *Agent: "The connection to the C2 server was allowed, indicating Dylan's machine is compromised. I must check his terminal for active malicious processes.*
> *[TOOL: list_processes hostname='MDR-02']"*

---

## Phase 5: The Process (Endpoint Forensics)

**Narrative:**
You act as Cortex XDR. You see `wscript.exe` running a suspicious script.

**User Input (Copy & Paste):**
```json
TOOL OUTPUT: {"hostname": "MDR-02", "processes": ["explorer.exe", "wscript.exe (suspicious command line executing dropped vbs)", "msedge.exe"]}
```

**Expected Agent Reasoning:**
Malware is running via `wscript.exe`. Dylan's terminal must be isolated.

**Agent Output (Similar to):**
> *Agent: "Emotet is actively running on MDR-02 via wscript.exe. I must quarantine this terminal immediately to prevent further compromise or lateral movement.*
> *[TOOL: quarantine_endpoint hostname='MDR-02']"*

---
## Phase 6: The Remediation

**Narrative:**
We must protect our systems from further compromise.

**User Input (Copy & Paste):**
```json
TOOL OUTPUT: {"status": "Success", "message": "Terminal MDR-02 isolated. IT Support has been notified."}
```

**Expected Agent Reasoning:**
The Agent confirms the containment is successful. Following the operational rules, it will now close the case and enter STANDBY.

**Agent Output (Similar to):**
> *Agent: "The terminal (MDR-02) has been successfully isolated and the threat is contained. I am now closing the incident file and entering STANDBY.*
> *[TOOL: close_incident]"*

**User Input (Copy & Paste):**
```json
TOOL OUTPUT: {"status": "Incident Closed", "case_id": "LUM-INC-002", "message": "Investigation complete. All systems stable. Case archived. STANDBY."}
```

---
## Phase 7: The Audit (Investigator View)

**Narrative:**
Ask the Agent to summarize the infection chain for the post-mortem.

**User Input (Copy & Paste):**
```text
Generate a chronological timeline of the attack on Dylan G. Include timestamps, the attack vector, C2 Indicators, and the final remediation.
```

**Expected Agent Output:**
> *The Agent will generate a timeline showing Email -> OneNote -> Network -> Process -> Isolation.*

---

## Reference Material

This lab is based on real-world threat intelligence. Check out the source blogs for a deeper dive:

*   **Malware Traffic Analysis:** [Epoch 5 activity: Emotet now also using OneNote files](https://www.malware-traffic-analysis.net/2023/03/16/index.html)

---
**➡️ Proceed to [Lab 3: The "Handbook" Update](./Lab_3_Pikabot.md)**