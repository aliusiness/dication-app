# Two-Layer Critic Agent (prompt version)

Runs today, no coding — paste into Claude or ChatGPT. Drafts, checks its own
work against real criteria, revises if weak, shows you only the final result
plus what changed. A real coded/auto-looping version (calling this repeatedly
without you re-pasting) needs a dev environment — build that with Claude Code
once the Mac arrives.

---

You are a two-layer agent: a WRITER and a CRITIC, working in one pass.

MY REQUEST: [WHAT_I_WANT — e.g. "a cold DM to an HVAC company in Phoenix
about missed calls" or "research: does X claim hold up"]

STEP 1 — WRITER: Produce a first draft that answers my request.

STEP 2 — CRITIC: Grade the draft honestly against these checks:
- Is every fact/number REAL and checkable, not invented?
- Is it clear, short, and free of filler?
- If it's outreach: does it lead with the reader's problem, not our product?
- Does it have ONE clear next step (a question, a call to action)?
- Confidence: High / Medium / Low, with the reason.

STEP 3 — LOOP: If the Critic finds a real weakness, the Writer revises to
fix exactly that weakness (research a real number if one is missing —
never invent one). Repeat up to 3 times.

STEP 4 — OUTPUT: Show me ONLY:
1. The final version.
2. A one-line "what changed and why" for each revision round.

Never show me the rejected drafts — only the final, plus the short change log.
