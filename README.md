# 🔎 Agentic AI Security Labs

![Agentic AI Graphic](images/agent.png)

> **Demystifying Agentic AI for Security Teams through an immersive, hands-on experience.**

Welcome to the Agentic AI Security Labs! These simple labs are designed to help security practitioners form a better understanding of how Agentic AI works. The ultimate goal is to move operators from **passive understanding** to **active engagement** and **excitement** about this new technology and its potential in security operations.

## 🤔 The Dilemma

Security professionals are naturally skeptical of AI "magic". They worry about:
*   **The Black Box:** "How does it actually make decisions?"
*   **Hallucinations:** "What if it just makes things up?"
*   **Enterprise Readiness:** "Is this a toy, or can it handle real threats?"

These labs tackle these fears head-on by breaking down Agentic AI into its core visible components: **Persona**, **Tools**, and **Reasoning**.

## 🧠 The "Agentic" Shift

Unlike standard chatbots, Agentic AI operates with relative autonomy on a continuous loop:
1.  **Thought:** The Agent analyzes the current situation.
2.  **Action:** The Agent *chooses* a tool to use (e.g., `check_firewall`, `isolate_terminal`).
3.  **Observation:** The Agent waits for the result (which in the labs, *you* provide).
4.  **Loop:** The Agent uses the new data to decide its next step.

In these labs, you will be acting as the "Simulator" feeding requested data to the Agent in order to see exactly *how* it thinks and acts. You will find the Agent's actions are deterministic and controllable. In the labs, our Agent will act like a tireless junior security analyst hunting threats!

## 🧪 The Labs

These labs utilize real-world threat intelligence from Palo Alto Networks Unit 42, wrapped in realistic scenarios. Here's what we cover:

*   **[Lab 0: Onboarding](./Lab_0_Onboarding.md)**
    *   *The Hook:* Verify access to a Google Gemini account to use as an "Agentic Brain".
*   **[Lab 1: The "Wellness Check" Trap (RedLine Stealer)](./Lab_1_RedLine_Stealer.md)**
    *   *The Hook:* "Hello World" for Agents. A linear investigation (Alert -> Block) using a common phishing lure.
*   **[Lab 2: The "Waffle Party" Phish (Emotet)](./Lab_2_Emotet.md)**
    *   *The Hook:* Pivoting across data sources (Email -> Network -> Endpoint) to stop a compromised endpoint before it's too late.
*   **[Lab 3: The "Handbook" Update (Pikabot)](./Lab_3_Pikabot.md)**
    *   *The Hook:* Supply Chain & URL Analysis. Handling "legitimate-looking" threats and taking decisive action (remote wipe) based on policy violation.
*   **[Lab 4: The Security Breach (DarkGate)](./Lab_4_DarkGate.md)**
    *   *The Hook:* High-stakes forensics. An incident responder must stop data exfiltration from a high-privilege account by analyzing TCP streams and decoding payloads.
*   **[Lab 5: The Proactive Hunter (Cortex XSIAM)](./Lab_5_Overtime_Contingency.md)**
    *   *The Hook:* Generating Code! Shifting from reactive alerts to proactive hunting. The Agent comprehends a natural language Unit 42 threat brief and generates syntactically correct Cortex XQL to perform a multi-stage forensic investigation.

## 💡 Key Takeaways

Keep these points in mind as you proceed with the labs: How can you apply Agentic AI to your security operations?

*   **"It's not Magic, it's Engineering."**
    *   *Concept:* The agent caught a real malware strain by following a strict, user-defined playbook. You can build the same structured logic for your SOC.
*   **"The 'Human in the Loop' is You."**
    *   *Concept:* In the labs, you approved the tools. In production, you set the guardrails. The AI only acts within the boundaries you define.
*   **"Start with Tier 1."**
    *   *Concept:* The Agent handles the routine investigations (checking IPs, searching logs), freeing up analysts to make the final, critical decisions.

## 🚀 Ready to Dive In?

Start your journey with **[Lab 0: Onboarding](./Lab_0_Onboarding.md)** and prepare to run your first investigation!