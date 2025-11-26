Step 1 — Baseline Prompt  

Prompt example:  
```markdown
Summarize this meeting
```


=================================================================================


Step 2 — Add Role (Who)  
Structure:  
```markdown
As a {role} {role experience or responsibility}.
{action}.
```

Prompt example:  
```markdown
As a Developer responsible for breaking down the work into implementable tasks, summarize the meeting from your perspective.
```

Optional instructions:  
- Rerun as a Mobile developer from a different team who needs to spot cross‑team dependencies.  
- Rerun as an Engineering Manager to highlight delivery risks and feature‑flag rollout.  
- Rerun as a PM or Designer to emphasize manager expectations, blocked‑vs‑hidden UX.  


=================================================================================


Step 3 — Add Structure & Intent (What and Why)  
Structure:  
```markdown
As a {role} {role experience or responsibility}.
You will use this summary to {intent}.
{action}.
{structure}.
```

Prompt example:  
```markdown
As a Developer responsible for breaking down the work into implementable tasks.

You will use this summary to plan and track the tasks you need to deliver from this project.

Produce a structured summary of this meeting with these sections:

1. Customer & Stakeholder Impact  
2. Technical Decisions & Constraints  
3. Risks, Blockers & Dependencies  
4. Owners & Task Breakdown  
5. Jira ticket drafts

```

Optional instructions:  

- In **Customer Impact**, mention how permission bugs affect trust and support tickets.  
- Highlight the top risks and open questions you heard.  
- End with 3–5 concrete follow‑ups.  


=================================================================================


Step 4 — Control Format & Length (How + How Much)  
Structure:  
```markdown
As a {role} {role experience or responsibility}.
You will use this summary to {intent}.
{action}.
{structure, format & length}.
```

Prompt example: 
```markdown
As a Developer responsible for breaking down the work into implementable tasks.

You will use this summary to turn the discussion into a concrete implementation plan and highlight any risky or cross‑team dependencies.

Summarize the meeting into these sections:

1. Customer & Stakeholder Impact — one paragraph, maximum 50 words.  
2. Technical Decisions & Constraints — a markdown table with columns: decision, constraint, impact; rows sorted by impact (highest first).  
3. Risks, Blockers & Dependencies — top‑level bullets, each with at most one short sub‑bullet for detail.  
4. Owners & Task Breakdown — one line per owner, in the format `{owner_name}[role]: {task_or_next_step}`.  
5. Jira ticket drafts:
 - Title: [{issue_type}] {issue_title}
 - Description: {Context} as a paragraph and {AC} in bullet points

```

Optional instructions:  
- Do one run where you return everything as a single JSON object with keys `customer_impact`, `decisions_tradeoffs`, `risks_blockers`, and `owners_next_actions`.  
- Do another run where you compress the whole summary into a single markdown table with columns: section, key_points, risk_level.  
- Do a final run formatted as a stand‑up style update with exactly four lines, each starting with a tag: `[Impact]`, `[Decisions]`, `[Risks]`, `[Owners]`.  


=================================================================================


Step 5 — Build Your Own
Prompt:  
```markdown
You are a [Role]...
```