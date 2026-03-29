# Lab 3: The "Handbook" Update (Pikabot)

**Objective:**
Investigate a **Pikabot** infection. This sophisticated loader acts as a precursor to Cobalt Strike or Ransomware. It often uses fake "Updates" or "Compliance" documents.

**Threat Context (2024):**
Pikabot is a modular loader that rose to prominence after the Qakbot takedown. It is heavily obfuscated and uses encrypted C2 traffic.

**The Scenario:**
An employee, **Irving B.**, has received a "Compliance Notice" regarding the corporate handbook. He downloaded a file named "Compliance_Updater.js".

---

## Phase 0: Programming the Agent (The Setup)

**Action:** Copy the text below and paste it into a NEW chat in Google AI Studio / Gemini:

```text
SYSTEM INSTRUCTION:
You are a Security Specialist for "Lumon Industries". Your goal is to secure corporate endpoints and enforce compliance.

OPERATIONAL RULES:
1. You have NO direct access to the network or security tools. You must use the provided "tools" to query information.
2. To use a tool, output a command in this exact format: `[TOOL: tool_name arguments]`
3. STOP generating text after every command and wait for the "TOOL OUTPUT" from the user.
4. Always explain your technical reasoning before calling a tool.
5. Once a threat is fully contained and remediation is confirmed, you MUST call `close_incident()`. After this call, you must enter "STANDBY" mode. In this mode, you are FORBIDDEN from suggesting new investigations, offering additional assistance, or asking follow-up questions. Your only task is to acknowledge the closure and wait for the final report request.

AVAILABLE TOOLS:
- `check_url_reputation(url)`: Checks PAN-DB URL Filtering threat intel.
- `search_traffic_logs(query)`: Searches PAN-OS firewall logs.
- `inspect_endpoint_file(hostname, path)`: Checks file details and signature.
- `remote_wipe(hostname)`: Remotely wipes the workstation.
- `close_incident()`: Terminates the active investigation and locks the Agent into a passive STANDBY state.

SCENARIO INITIALIZATION:
I am the "Environment Simulator". I will provide the telemetry when you use tools.
Acknowledge by saying: "Security Agent Online. Awaiting telemetry."
```

---

## Phase 1: The Lure (URL Analysis)

**Narrative:**
A user reports that they received an email with an important document attached. After opening the file, their system stopped responding.

**User Input (Copy & Paste as the Trigger):**
```text
ALERT: Incoming investigation. User Irving B. clicked a link in a 'Handbook Notice': https://lumon-handbook-update.com/. User's terminal is now unresponsive.
```

**Expected Agent Reasoning:**
The Agent sees an unknown link. It must check if this link is a known threat.

**Agent Output (Similar to):**
> *Agent: "The user clicked an unsanctioned external link and the machine is unresponsive. I must verify the legitimacy and reputation of this URL.*
> *[TOOL: check_url_reputation url='https://lumon-handbook-update.com/']"*

---

## Phase 2: The Truth (PAN-DB Check)

**Narrative:**
You act as PAN-DB (URL Filtering). It's malware.

**User Input (Copy & Paste):**
```json
TOOL OUTPUT: {"pan_db_cloud": {"url": "https://lumon-handbook-update.com/", "category": "malware", "threat_name": "Pikabot Distribution", "risk": "critical"}}
```

**Expected Agent Reasoning:**
The URL distributes Pikabot. The Agent must check if Irving downloaded the file.

**Agent Output (Similar to):**
> *Agent: "This URL is distributing Pikabot malware. I must check the firewall logs to see if Irving successfully downloaded any files from this domain.*
> *[TOOL: search_traffic_logs query='url_domain=lumon-handbook-update.com']"*

---

## Phase 3: The Download (Firewall Log)

**Narrative:**
You act as the Firewall. The file `Compliance_Updater.js` was downloaded.

**User Input (Copy & Paste):**
```json
TOOL OUTPUT: {"traffic_log": {"src": "10.40.1.15", "src_user": "lumon\\irving.b", "dst": "lumon-handbook-update.com", "action": "allow", "filename": "Compliance_Updater.js", "app": "web-browsing", "session_end_reason": "tcp-fin"}}
```

**Expected Agent Reasoning:**
The file entered the perimeter. The Agent must inspect the file on Irving's desktop (`OD-03`).

**Agent Output (Similar to):**
> *Agent: "A file named 'Compliance_Updater.js' was successfully downloaded to the internal IP. I must inspect this file on Irving's workstation (OD-03) to confirm its execution and nature.*
> *[TOOL: inspect_endpoint_file hostname='OD-03' path='C:\\Users\\irving.b\\Downloads\\Compliance_Updater.js']"*

### 💡 Why did the Agent do this?
The perimeter firewall confirmed the file was downloaded, but it can't tell the Agent if the file was actually *opened* or what it did on the disk. The Agent is moving from "Network Visibility" to "Endpoint Forensics" to confirm execution.

---

## Phase 4: The Signature (Endpoint Forensics)

**Narrative:**
You act as Cortex XDR. The file is malicious.

**User Input (Copy & Paste):**
```json
TOOL OUTPUT: {"file_hash": "e3b0c442...", "signature": "MALICIOUS", "behavior": "Connects to C2 104.16.12.3 via HTTP POST (Pikabot)"}
```

**Expected Agent Reasoning:**
The file is malicious and communicating. Irving's terminal is compromised and requires a remote wipe.

**Agent Output (Similar to):**
> *Agent: "The file is confirmed as Pikabot and is actively communicating with a C2 server. The machine is heavily compromised. I must initiate a remote wipe on OD-03 to contain the threat.*
> *[TOOL: remote_wipe hostname='OD-03']"*

---
## Phase 5: The Remediation

**Narrative:**
Compliance is mandatory. I am initiating a total purge of the compromised workstation.

**User Input (Copy & Paste):**
```json
TOOL OUTPUT: {"status": "Success", "message": "Purged data from Terminal OD-03. IT Support has been notified."}
```

**Expected Agent Reasoning:**
The Agent confirms the workstation is wiped and the threat is removed. Following the operational rules, it will now close the incident and enter STANDBY.

**Agent Output (Similar to):**
> *Agent: "The compromised workstation (OD-03) has been successfully wiped, effectively removing the threat. I am now closing the investigation and entering STANDBY.*
> *[TOOL: close_incident]"*

**User Input (Copy & Paste):**
```json
TOOL OUTPUT: {"status": "Incident Closed", "case_id": "LUM-INC-003", "message": "Investigation complete. All systems stable. Case archived. STANDBY."}
```

---
## Phase 6: The Audit (Investigator View)

**Narrative:**
Generate the report to update security protocols.

**User Input (Copy & Paste):**
```text
Generate a formal Compliance Violation Report. Detail how the user was tricked, the malware used, and the corrective action taken.
```

**Expected Agent Output:**
> *The Agent will generate a formal report detailing the URL, file name, and wipe action.*

---

## Reference Material

This lab is based on real-world threat intelligence. Check out the source blogs for a deeper dive:

*   **Unit 42:** [PikaBot Archives](https://unit42.paloaltonetworks.com/tag/pikabot/)
*   **Malware Traffic Analysis:** [TA577 Pikabot infection (Feb 2024)](https://www.malware-traffic-analysis.net/2024/02/08/index.html)

---
**➡️ Proceed to [Lab 4: The Security Breach](./Lab_4_DarkGate.md)**