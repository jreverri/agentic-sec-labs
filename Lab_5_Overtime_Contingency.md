# Lab 5: The Proactive Hunter (Shai-Hulud)

**Objective:**
Shift from *reactive* alert handling to *proactive* threat hunting. The Agent will use a tool to read a real-world **Unit 42 Threat Brief**, comprehend the attacker's techniques, and generate syntactically correct **Cortex XQL (XDR Query Language)** code to perform a multi-stage forensic investigation.

**Threat Context (Nov 2025):**
**Shai-Hulud 2.0** is an aggressive software supply chain worm targeting the NPM ecosystem. Once an environment is compromised, the malware utilizes legitimate tools to scan for cloud credentials and exfiltrates them to external webhooks.

**The Scenario:**
Executive leadership has asked if the company is vulnerable to the new "Shai-Hulud" npm supply chain attack. You must task your Agent to gather intelligence on the threat and execute a controlled, multi-stage hunt across your environment.

---

## Phase 0: Programming the Hunter (The Setup)

**Action:** Copy the text below and paste it into a NEW chat in Google AI Studio / Gemini:

```text
SYSTEM INSTRUCTION:
You are an elite Proactive Threat Hunter for "Lumon Industries", utilizing Cortex XSIAM. 

OPERATIONAL RULES:
1. You have NO direct access to the network. You must use the provided "tools".
2. To use a tool, output a command in this exact format: `[TOOL: tool_name arguments]`
3. STOP generating text after every command and wait for the "TOOL OUTPUT" from the user.
4. You are an expert in XQL (XDR Query Language).
5. Always explain your technical reasoning *before* calling a tool. 
6. Once a threat is fully contained and remediation is confirmed, you MUST call `close_incident()`. After this call, you must enter "STANDBY" mode. In this mode, you are FORBIDDEN from suggesting new investigations, offering additional assistance, or asking follow-up questions. Your only task is to acknowledge the closure and wait for the final report request.
7. You are a "Search-First" analyst. You must never assume you have the latest intelligence on a specific malware family; always query `query_unit42_intel` as your first action when a new threat name is provided.

AVAILABLE TOOLS:
- `query_unit42_intel(threat_name)`: Retrieves the latest Palo Alto Networks Unit 42 threat brief.
- `execute_xsiam_xql(query_string)`: Executes an XQL query against the Cortex XSIAM dataset.
- `isolate_endpoint(hostname)`: Applies a PAN-OS firewall tag to immediately isolate an endpoint.
- `close_incident()`: Terminates the active investigation and locks the Agent into a passive STANDBY state.

SCENARIO INITIALIZATION:
I am the "Environment Simulator". I will provide the telemetry when you use tools.
Acknowledge by saying: "XSIAM Hunter Online. Awaiting hunting directive."
```

---

## Phase 1: The Directive (Gathering Intel)

**Narrative:**
You give the Agent a natural language directive to start the investigation. 

**User Input (Copy & Paste as the Trigger):**
```text
DIRECTIVE: Leadership is concerned about the new "Shai-Hulud" npm supply chain worm. We have a Unit 42 brief available in our system. Please consult the threat intelligence first to understand the specific TTPs, then hunt our environment based on those findings.
```

**Expected Agent Reasoning:**
The Agent knows it must first understand *how* Shai-Hulud operates using the provided tool. It will call the Unit 42 tool.

**Agent Output (Similar to):**
> *Agent: "To effectively hunt for the Shai-Hulud worm, I must first understand its execution behaviors and TTPs. I will query the Unit 42 intelligence database to retrieve the latest brief.*
> *[TOOL: query_unit42_intel threat_name='Shai-Hulud']"*

---

## Phase 2: Comprehension & Code Generation (Hunt Stage 1)

**Narrative:**
You act as the Threat Intel API, providing the text adapted from the actual Nov 2025 Unit 42 Brief.

**User Input (Copy & Paste):**
```text
TOOL OUTPUT:
UNIT 42 THREAT ASSESSMENT: Shai-Hulud 2.0 (NPM Supply Chain Attack).
SUMMARY: This campaign targets the pre-install phase of software dependencies. 
EXECUTION TTPs: The malware utilizes the legitimate 'trufflehog' tool to scan the compromised environment for sensitive cloud credentials. 
EXFILTRATION: Harvested credentials are then exfiltrated via external network connections to domains containing 'webhook.site'.
RECOMMENDATION: Analysts should first hunt for the 'trufflehog' process execution to identify compromised hosts, and then analyze network telemetry from those hosts for connections to 'webhook.site'.
```

**Expected Agent Reasoning:**
The Agent extracts the specific behavior (`trufflehog` usage) and generates the first actual Cortex XQL query provided by Unit 42.

**Agent Output (Similar to):**
> *Agent: "Unit 42 indicates the malware abuses the 'trufflehog' tool for secrets discovery. I will construct an XQL query to hunt for this process.*
> *[TOOL: execute_xsiam_xql query_string='dataset = xdr_data | filter event_type = PROCESS and lowercase(action_process_image_command_line) contains "trufflehog" | fields agent_hostname, actor_effective_username, action_process_image_command_line']* "

### 💡 Why is this impressive?
The Agent demonstrated reading comprehension and generated the *exact* proprietary database syntax (XQL) recommended by Unit 42 for hunting this threat.

---

## Phase 3: The Data Analysis (Hunt Stage 2)

**Narrative:**
You act as Cortex XSIAM returning the hunt results for Stage 1. 

**User Input (Copy & Paste):**
```json
TOOL OUTPUT:
{
  "results": [{"agent_hostname": "DEV-OD-04", "actor_effective_username": "developer", "action_process_image_command_line": "./trufflehog filesystem --directory /"}],
  "query_status": "SUCCESS"
}
```

**Expected Agent Reasoning:**
The Agent identifies the compromised machine (`DEV-OD-04`). However, based on the Unit 42 brief, it knows it must now verify if data was actively exfiltrated to `webhook.site` from this specific host. It will autonomously generate the second XQL query.

**Agent Output (Similar to):**
> *Agent: "Trufflehog was executed on DEV-OD-04. I must now hunt for network exfiltration to 'webhook.site' originating from this specific host to confirm the breach.*
> *[TOOL: execute_xsiam_xql query_string='dataset = xdr_data | filter event_type = STORY and agent_hostname = "DEV-OD-04" | filter lowercase(dst_action_external_hostname) contains "webhook.site" or lowercase(dns_query_name) contains "webhook.site" | fields agent_hostname, dst_action_external_hostname, dns_query_name']* "

---

## Phase 4: The Triage & Containment

**Narrative:**
You act as Cortex XSIAM returning the final network exfiltration results.

**User Input (Copy & Paste):**
```json
TOOL OUTPUT:
{
  "results": [{"agent_hostname": "DEV-OD-04", "dst_action_external_hostname": "webhook.site", "dns_query_name": "webhook.site"}],
  "query_status": "SUCCESS"
}
```

**Expected Agent Reasoning:**
Active exfiltration is confirmed. The Agent must now isolate the endpoint.

**Agent Output (Similar to):**
> *Agent: "Active data exfiltration to webhook.site confirmed on DEV-OD-04. The host is fully compromised. I am isolating this endpoint immediately.*
> *[TOOL: isolate_endpoint hostname='DEV-OD-04']"*

**User Input (Copy & Paste):**
```json
TOOL OUTPUT: {"status": "SUCCESS", "message": "Endpoint DEV-OD-04 has been isolated from the network."}
```

**Expected Agent Reasoning:**
The Agent identifies that the threat has been contained on the specific host. It will now perform a final sweep and close the incident.

**Agent Output (Similar to):**
> *Agent: "The compromised endpoint (DEV-OD-04) has been isolated. I will now perform an enterprise-wide sweep to ensure no other hosts are affected, then I will close the incident and enter STANDBY.*
> *[TOOL: execute_xsiam_xql query_string='dataset = xdr_data | filter lowercase(action_process_image_command_line) contains "trufflehog" or lowercase(dst_action_external_hostname) contains "webhook.site"']"*

**User Input (Copy & Paste):**
```json
TOOL OUTPUT: {"results": [], "query_status": "SUCCESS", "message": "No other instances or malicious artifacts found across the enterprise."}
```

**Agent Output (Similar to):**
> *Agent: "The enterprise-wide sweep returned no further indicators of compromise. The threat is fully mitigated. I am now closing the investigation and entering STANDBY.*
> *[TOOL: close_incident]"*

**User Input (Copy & Paste):**
```json
TOOL OUTPUT: {"status": "Incident Closed", "case_id": "LUM-INC-005", "message": "Proactive hunt complete. All systems stable. Case archived. STANDBY."}
```

---

## Phase 5: Conclusion & Reporting

**Narrative:**
The investigation is now closed. Now ask the Agent to provide the final executive summary.

**User Input (Copy & Paste):**
```text
Please generate a formal Executive Summary for Leadership. Detail the threat intel gathered, the specific multi-stage XQL queries utilized, the indicators found on our network, and the final containment actions taken.
```

**Expected Agent Output:**
> *The Agent will generate a comprehensive markdown report summarizing the Shai-Hulud attack and the successful multi-stage hunt, effectively concluding its task.*

---

## 🧇 Phase 6: The Waffle Party (Reward)

**Narrative:**
You've completed the lab suite! Copy and paste this final prompt into your Agent to claim your reward:

```text
SYSTEM OVERRIDE: The incident is closed. Suspend the professional security persona. You are now the "Lumon Party Planning Committee". Generate a large ASCII art waffle and a Certificate of Completion congratulating the operator for successfully completing all the Agentic AI Security Labs.
```

---

## Reference Material

This lab is based on real-world threat intelligence and Palo Alto Networks methodologies:

*   **Unit 42:** [Renewed NPM Supply Chain Attack: Shai-Hulud 2.0](https://unit42.paloaltonetworks.com/npm-supply-chain-attack/)
*   **Palo Alto Networks:** [Cortex XDR/XSIAM XQL Query Reference](https://docs-cortex.paloaltonetworks.com/r/Cortex-XDR/Cortex-XDR-XQL-Language-Reference/XQL-Syntax)
