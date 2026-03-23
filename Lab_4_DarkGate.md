# Lab 4: The Security Breach (DarkGate)

**Objective:**
Act as a **Security Analyst** to investigate a **DarkGate** infection where highly sensitive data is being exfiltrated. This is a pure **Network Forensics** and **Data Loss Prevention (DLP)** lab.

**Threat Context (2024/2025):**
DarkGate is a Malware-as-a-Service (MaaS) that provides advanced exfiltration capabilities (Reverse Shells, Hidden VNC).

**The Scenario:**
An employee from the Security Office, **Mr. Milchick**, is the source of the breach. His terminal is sending large amounts of data to an unknown location.

---

## Phase 0: Programming the Agent (The Setup)

**Action:** Copy the text below and paste it into a NEW chat in Google AI Studio / Gemini:

```text
SYSTEM INSTRUCTION:
You are a Lead Incident Responder for "Lumon Industries". Your goal is to identify and stop unauthorized data exfiltration.

OPERATIONAL RULES:
1. You have NO direct access to the network or security tools. You must use the provided "tools" to query information.
2. To use a tool, output a command in this exact format: `[TOOL: tool_name arguments]`
3. STOP generating text after every command and wait for the "TOOL OUTPUT" from the user.
4. Always explain your technical reasoning before calling a tool.

AVAILABLE TOOLS:
- `get_threat_log(alert_id)`: Reads the PAN-OS threat alert.
- `inspect_tcp_stream(src_ip, dest_port)`: Analyzes a specific network stream.
- `decode_payload(stream_id)`: Decodes raw exfiltrated data.
- `terminate_session(session_id)`: Forcefully kills a network session.

SCENARIO INITIALIZATION:
I am the "Environment Simulator". I will provide the telemetry when you use tools.
Acknowledge by saying: "Incident Responder Online. Awaiting telemetry."
```

---

## Phase 1: The Alert (PAN-OS Threat Log)

**Narrative:**
Palo Alto Networks Threat Prevention running on the firewalls has flagged critical C2 traffic.

**User Input (Copy & Paste as the Trigger):**
```text
ALERT: The perimeter has detected unauthorized Spyware activity from SECURITY-PC-01 (Milchick). Threat ID: 86902.
```

**Expected Agent Reasoning:**
The Agent sees a Spyware alert from the Security Office. It must inspect the log.

**Agent Output (Similar to):**
> *Agent: "Spyware activity originating from a security terminal is a critical alert. I must examine the Threat Log details immediately.*
> *[TOOL: get_threat_log alert_id='86902']"*

---

## Phase 2: The Log (Threat Detail)

**Narrative:**
You act as the Threat Log. It's DarkGate C2 traffic to port 8080.

**User Input (Copy & Paste):**
```json
TOOL OUTPUT: {"threat_log": {"subtype": "spyware", "threat_name": "DarkGate.Gen Command and Control Traffic", "src": "10.99.1.5", "src_user": "lumon\\milchick.s", "dst": "185.246.221.15", "dst_port": "8080", "severity": "critical"}}
```

**Expected Agent Reasoning:**
The Agent confirms DarkGate C2. It sees the destination port (`8080`). It must inspect the actual *content* of the stream.

**Agent Output (Similar to):**
> *Agent: "DarkGate C2 traffic is confirmed. The host is compromised. I must inspect the TCP stream to determine if data is actively being exfiltrated.*
> *[TOOL: inspect_tcp_stream src_ip='10.99.1.5' dest_port='8080']"*

---

## Phase 3: The Payload (Stream Forensics)

**Narrative:**
You act as Wireshark. You see a large (5.2MB) encrypted ZIP file leaving the network.

**User Input (Copy & Paste):**
```json
TOOL OUTPUT: {"stream_id": "TCP-4421", "size": "5.2MB", "content_type": "encrypted_zip", "dest_country": "Russia"}
```

**Expected Agent Reasoning:**
5MB of data is leaving. It's encrypted. The Agent must decode it to assess the damage.

**Agent Output (Similar to):**
> *Agent: "A large encrypted ZIP file is being exfiltrated. I must decode this payload to identify the stolen files and assess the impact.*
> *[TOOL: decode_payload stream_id='TCP-4421']"*

### 💡 Why did the Agent do this?
In a Data Loss Prevention (DLP) scenario, knowing *that* data left is only half the battle. To assess the impact, the responder must know *what* data left. The Agent is autonomously attempting to decrypt/decode the stream to identify the specific sensitive files stolen.

---

## Phase 4: The Damage (Data Loss)

**Narrative:**
You act as the Decoder. The files are critical secrets.

**User Input (Copy & Paste):**
```json
TOOL OUTPUT: {"status": "Decoded", "files_detected": ["Employee_List.xlsx", "Security_Recordings.mp4", "Executive_Notes.docx"]}
```

**Expected Agent Reasoning:**
The Agent sees critical documents. The session must be killed immediately.

**Agent Output (Similar to):**
> *Agent: "Highly sensitive corporate data is being stolen. This is a severe data breach. I must terminate this session immediately to stop the exfiltration.*
> *[TOOL: terminate_session session_id='TCP-4421']"*

**User Input (Copy & Paste):**
```json
TOOL OUTPUT: {"status": "Session Terminated", "message": "Connection to 185.246.221.15 severed. Security leadership notified."}
```

---

## Phase 5: The Audit (Investigator View)

**Narrative:**
Security Leadership demands a report.

**User Input (Copy & Paste):**
```text
Generate a Breach Impact Assessment for executive leadership. Detail the data lost, the insider responsible, and the network indicators observed.
```

**Expected Agent Output:**
> *The Agent will generate a critical incident report.*

---

## Reference Material

This lab is based on real-world threat intelligence. Check out the source blogs for a deeper dive:

*   **Unit 42:** [DarkGate: Dancing the Samba With Alluring Excel Files](https://unit42.paloaltonetworks.com/darkgate-malware-uses-excel-files/)
*   **Malware Traffic Analysis:** [DarkGate Infection (March 2024)](https://www.malware-traffic-analysis.net/2024/03/19/index.html)

---
**➡️ Proceed to [Lab 5: The "Overtime Contingency" (Capstone)](./Lab_5_Overtime_Contingency.md)**