# Lab 5: The Proactive Hunter (Cortex XSIAM)

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

AVAILABLE TOOLS:
- `query_unit42_intel(threat_name)`: Retrieves the latest Palo Alto Networks Unit 42 threat brief.
- `execute_xsiam_xql(query_string)`: Executes an XQL query against the Cortex XSIAM dataset.
- `isolate_endpoint(hostname)`: Applies a PAN-OS firewall tag to immediately isolate an endpoint.

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
DIRECTIVE: Leadership is concerned about the new "Shai-Hulud" npm supply chain worm. Please gather intelligence on its execution behaviors and hunt our environment for them.
```

**Expected Agent Reasoning:**
The Agent knows it must first understand *how* Shai-Hulud operates. It will call the Unit 42 tool.

**Agent Output (Similar to):**
> *Agent: "I need to understand the TTPs of the Shai-Hulud worm before I can hunt for it. I will query Unit 42.*
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

---

## Phase 5: Conclusion & Reporting

**Narrative:**
A proactive Agent doesn't stop at just one host. It will autonomously decide to sweep the rest of the enterprise or perform further forensics on the isolated host. To conclude the lab on a high note, we will provide the final "All Clear" data AND the command for the final report in a single step. 

> **Note:** By providing the telemetry and the reporting command in a single message, you are effectively "closing the ticket," forcing the Agent to stop hunting and provide the final executive summary.

**Expected Agent Output (Similar to):**
> *Agent: "DEV-OD-04 is contained. I will now perform an enterprise-wide sweep to ensure no other endpoints exhibit this behavior.*
> *[TOOL: execute_xsiam_xql query_string='...']* "

**User Input (Copy & Paste):**
```text
TOOL OUTPUT: {"results": [], "query_status": "SUCCESS", "message": "No other instances or malicious artifacts found across the enterprise."}

The investigation is now closed. Please generate a formal Executive Summary for Leadership. Detail the threat intel gathered, the specific multi-stage XQL queries utilized, the indicators found on our network, and the final containment actions taken.
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
