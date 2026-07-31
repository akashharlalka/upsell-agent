Upsell Agent — Engineering Notes
This is engineering-specific context that doesn't belong inside the agent's own instructions. For the actual behavior rules — persona, topics, response structure, entitlement handling, standing rules — see:
•	agent-instructions-mvp.md — what the MVP agent should build against right now.
•	agent-instructions-vision.md — the full reference version (all semantics, all topics), for later.
Those two files are the single source of truth for agent behavior. This document exists so that source of truth stays in one place instead of getting re-explained here in different words and quietly drifting out of sync with it — which is exactly what happened the first time this doc existed as a standalone rulebook.

The one thing worth internalizing before reading the instruction docs
Nothing in the instructions is a fixed checklist of "this type of question gets these response parts." Every part — Adjacent value, the UI card, Related questions — is included only if it genuinely helps with the specific message in front of the agent, not because messages like it usually get one. A technically-allowed CTA that doesn't fit the moment (e.g. pitching a trial to someone already mid-trial) still gets suppressed. If a generated response looks "incomplete" compared to some expected pattern, check whether that's actually a judgment call working correctly before assuming it's a bug.

What's a spec vs. what's an experiment
A planning slide or worked-example table showing "this case gets these parts" is useful as a source of test cases and a coverage check — did we forget a whole component type exists — but it is explicitly not something the agent's instructions are built to satisfy line-for-line. If a test case and the agent's actual judgment disagree, that's not automatically a bug in the agent; it's a prompt to ask whether the judgment is right. Treat any such table as a hypothesis to run against the agent, not a checklist to backfill into its instructions.

What the tool actually is (and isn't)
The tool is a content-authoring and reference environment, not the product UI and not the deliverable itself.
•	The Prompt bank holds worked examples — an utterance paired with a response that correctly applies the current instructions. Treat each row as "here's what a correct answer to this looks like," not literal copy to ship verbatim.
•	The Response Output tab mirrors the instruction docs in the tool's own words — if the tool and the instruction docs ever disagree, the instruction docs win, and the tool should be updated to match.
•	The real deliverable for engineering is the exported spreadsheet (utterance → topic → response logic → entitlement), not the HTML tool itself.
•	Two versions exist: MVP (Informational semantic only, 4 topics) and Vision (the full reference version — all semantics, plus an Out-of-domain topic). Build against MVP only, right now.

Entitlement — the most common source of a wrong-looking response
Every capability has a status (Free, Premium-tool-tokens, Premium-plan, or Unclassified) that directly controls whether and how the UI card appears. If a generated response's card looks off, check the entitlement table before assuming the response logic is wrong — a missing or incorrect entitlement status is the most common actual cause.
Row hygiene: when a single verb covers multiple distinct actions with different entitlement (e.g. "delete" applies to a comment, a page, a line of text, and a signature — free, paid, paid, paid respectively), don't fold them into one row with a pile of aliases. Let each discrete action own its own row. An overloaded row with ambiguous aliases is exactly what causes the entitlement matcher to guess wrong — it's a data-modeling problem, not something a smarter matching algorithm can fully paper over.

Deferred features — designed, not built, don't reintroduce without a decision
One thing was fully designed and then explicitly scoped out of MVP by engineering. It isn't in agent-instructions-mvp.md anymore — removed from a doc the agent actually reads, since an agent can't selectively ignore a documented behavior it's told to produce. The design work isn't lost; it's just parked here until picked back up.
Form-mode UI card. A short tap-through form (asking team size, use case, or budget) as an alternative to a static CTA, for cases where one specific missing piece of information would meaningfully change the recommendation — a cold start, or an ambiguous plan-fit question. In MVP, this need is instead met by asking the same question conversationally in the Direct answer/Adjacent value text, with a default static CTA still supplied.

Repeat-suppression card rule — a real infrastructure gap, not just a content decision
agent-instructions-mvp.md now says the UI card is suppressed if the user re-probes the same specific capability/fact they just asked about, with a third consecutive ask resetting it back on. That's a content decision the team has agreed to. What it actually needs to run, though, is a persistent counter across at least 3 turns — the tool's current conversation memory (lastTurn in the JS) only tracks the single most recent turn, not a running count of consecutive same-entity asks.
This is the same category of open item as the Try-on-sample-PDF card (also pending engineering feasibility review) — don't assume this rule is live just because it's documented. If the runtime can't track a multi-turn counter yet, the safe fallback is the same pattern used elsewhere in this doc: default to showing the card every time until the counter infrastructure exists, rather than silently guessing at repeat-status.

Card-type system — this is new, and larger than the old single "UI card" concept
The UI card is no longer one thing with a present/absent toggle — it's now a family of distinct visual components (App-level, Tool-based, Try-1-tool-free, Trial timeline, plus Try-on-sample-PDF and Credits-based still being designed), each with its own trigger condition and its own CTA label. See the Card type table in agent-instructions-mvp.md's UI card section for the current mapping. Two things worth flagging to whoever builds this:
•	The exact copy for Try-1-tool-free and Try-on-sample-PDF's action labels is still pending — "Start free trial" is deliberately wrong for those two, but nothing's been finalized to replace it yet.
•	A "Learn more" / "See all plans" CTA type has been mentioned as a likely future addition — not designed yet, just flagged so it isn't a surprise when it arrives.
Related questions (Forward / Deeper / Wider) is no longer deferred — restored to full scope in agent-instructions-mvp.md per explicit direction. If it gets scoped out again in the future, this is the section it should come back to.
