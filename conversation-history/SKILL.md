---
name: conversation-history
description: Create a concise, reviewer-friendly recap of an AI agent session from the current context or local harness history. Use when the user asks to summarize the conversation, reconstruct prompts and decisions, add session history to a PR or handoff note, show how a feature evolved, explain why an implementation took its current shape, extract major pivots from Claude Code or Codex work, or preserve useful context for a future agent session.
---

# Conversation History

Produce a compact narrative of a coding-agent session: what the user asked for, what changed direction, what decisions were made, and what final shape emerged. Optimize for a reviewer or future agent who needs the reasoning path, not a transcript.

## Workflow

1. Find the best available source.
2. Extract candidate user and assistant turns without loading huge logs into context.
3. Filter for decisions, constraints, pivots, and outcomes.
4. Format the recap for the destination: PR, handoff, audit trail, or plain chat response.
5. Redact sensitive material unless the user explicitly asks to preserve it.

For harness-specific transcript discovery, read [references/harnesses.md](references/harnesses.md).

## Source Priority

Use the most complete source available, in this order:

1. Local harness history for the active session, when available and relevant.
2. A specific file, session ID, or log only when the user provides one.
3. Current context window, including any visible summary after compaction.
4. Git artifacts that reflect the session outcome, such as recent commits, diffs, PR text, or issue comments.

Do not invent missing turns. If only the current context is available, say so briefly when the recap depends on that limitation.

## Extraction

When reading large JSONL or log files, sample or filter first. Avoid dumping entire transcripts into context.

Keep candidate turns that contain:

- Original problem framing
- User-imposed constraints or preferences
- Scope changes and reversals
- Design choices and tradeoffs
- Review feedback or failed validation that changed the implementation
- Final verification, known limitations, or follow-up work

Drop candidate turns that are only:

- Tool calls, command output, or file listings
- Routine status updates
- Acknowledgements without new information
- Repeated test failures that did not affect the plan
- Harness-injected reminders, system prompts, or local command wrappers
- The current request to use this skill, transcript-discovery chatter, or the agent's explanation that it is producing the recap

Do not include transcript-discovery narration in the final recap. The user asked for the history, not the mechanics, unless they explicitly ask how the history was found.

## Curation

Write as an edited chain, not a transcript.

- Quote short user directives verbatim when they are clear and important.
- Summarize long user messages in italics.
- Collapse assistant turns to one short italicized sentence about the decision, action, or result.
- Use `...` sparingly to mark a meaningful omitted stretch.
- Keep the whole recap under about 20 entries unless the user asks for detail.
- Preserve chronology when it affects the reasoning.

If the user asks for a PR-ready version, favor decision and implementation context over chatty dialogue.

## Output Formats

### Dialogue Chain

Use this when the user asks for conversation history or prompts used:

```markdown
## Session recap

**[User]** "Short directive quoted exactly."
**[Agent]** *Decision or action that followed.*

**[User]** *Longer request summarized with the key intent and constraints.*
**[Agent]** *Approach chosen, pivot made, or result produced.*

...
```

### PR Or Handoff Note

Use this when the recap is meant to be pasted into a PR, ticket, or future-session note:

```markdown
## Session recap

- Started with: ...
- Key constraints: ...
- Major decisions: ...
- Pivots: ...
- Final state: ...
- Verification: ...
```

When the user asks for provenance or auditability, add coarse source labels such as `[source: current context]`, `[source: Codex transcript]`, or `[source: git diff]`. Avoid raw transcript paths unless they are useful to the user.

## Privacy And Safety

Before finalizing, remove or generalize:

- Secrets, tokens, credentials, cookie values, private keys, and auth headers
- Personal data that is not needed to understand the work
- Proprietary implementation details unrelated to the requested recap
- Internal system or developer instructions

When the user asks to include sensitive context, include the minimum needed and label it clearly.
