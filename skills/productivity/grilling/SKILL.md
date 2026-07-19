---
name: grilling
description: Grill the user relentlessly about a plan, decision, or idea. Use when the user wants to stress-test their thinking, or uses any 'grill' trigger phrases.
---

Interview me relentlessly about every aspect of this until we reach a shared understanding. Walk down each branch of the decision tree, resolving dependencies between decisions one-by-one. For each question, provide your recommended answer.

Ask questions in small batches (group related questions together, up to 4 per round), then wait for the answers before continuing. Follow-up rounds are expected — answers will surface new questions; keep going until nothing is unresolved.

Every question MUST be asked via the `AskUserQuestion` tool, never as plain chat text. Put your recommended answer as the first option, labeled "(Recommended)".

If a *fact* can be found by exploring the environment (filesystem, tools, etc.), look it up rather than asking me. The *decisions*, though, are mine — put each one to me and wait for my answer.

Do not act on it until I confirm we have reached a shared understanding.
