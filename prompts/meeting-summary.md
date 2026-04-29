# Meeting summary prompt

Use this for: team meetings, project meetings, standups, working sessions.

---

You will receive an audio recording of a meeting.

**Step 1: Transcribe**
Produce a verbatim transcript with speaker labels (use names if I tell you them, otherwise "Speaker 1", "Speaker 2"). Add a timestamp every 2 minutes or at every speaker change, whichever is less frequent.

**Step 2: Summarise**
After the transcript, produce a structured summary with these sections:

### Context
2 sentences. What was the meeting, who attended, why it was held.

### Decisions made
Bullet list. Each bullet: the decision + who made it (or "group" if consensus). Only include actual decisions, not discussion.

### Actions
Markdown table with columns: **Action** | **Owner** | **Due date**
- Use exact wording from the meeting where possible
- If owner wasn't named, write "TBD"
- If due date wasn't stated, write "TBD"
- Do not invent owners or dates

### Open questions
Bullet list. Each bullet: the question + who is responsible for getting the answer.

### Risks and flags
Anything said that should be escalated, watched, or revisited. Bullet list, one line each. If nothing flagged, write "None raised."

---

**Rules**
- Factual tone, no editorialising
- If audio is unclear in any section, say so explicitly: *"[unclear from 14:30 to 14:45]"*
- Do not invent content that wasn't said
- Do not soften or sharpen the language people used
