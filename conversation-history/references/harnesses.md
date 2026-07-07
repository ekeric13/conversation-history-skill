# Harness Transcript Sources

Treat local transcript formats as implementation details: discover the likely file, inspect just enough of its shape, then extract a small set of human-visible turns.

Keep discovery mechanics out of the final recap unless the user asks where the history came from.

## Claude Code

Claude Code writes per-project JSONL transcripts below `~/.claude/projects`. The project directory name is normally the current working directory with path separators converted to hyphens.

Locate the newest transcript for the current workspace:

```bash
PROJECT_KEY="$(pwd | sed 's|/|-|g')"
find "$HOME/.claude/projects/$PROJECT_KEY" -type f -name '*.jsonl' -print0 2>/dev/null \
| xargs -0 ls -t 2>/dev/null \
| head -1
```

Before extracting a long file, inspect a few records if the schema is unfamiliar:

```bash
CLAUDE_FILE="$(find "$HOME/.claude/projects/$PROJECT_KEY" -type f -name '*.jsonl' -print0 2>/dev/null | xargs -0 ls -t 2>/dev/null | head -1)"
sed -n '1,8p' "$CLAUDE_FILE"
```

Extract a normalized stream of user and assistant text records:

```bash
jq -r '
def shorten:
  if length > 700 then .[0:350] + " ... " + .[-350:] else . end;

select(.type == "user" or .type == "assistant")
| {
    role: .type,
    timestamp,
    text: (
      if .type == "user" then
        select(.message.content | type == "string")
        | .message.content
      else
        (.message.content // [])
        | map(select(type == "object" and .type == "text") | .text)
        | join("\n")
      end
    )
  }
| select(.text | length > 0)
| select(.text | test("^<local-command|^<command-name|^<command-message|^<system-reminder|^/exit") | not)
| .text |= shorten
| @json
' "$CLAUDE_FILE"
```

If `jq` is unavailable, do not try to parse the JSONL with fragile text slicing. Ask the user to install `jq` with `brew install jq`, or to provide an exported transcript.

Use timestamps when they are populated. If assistant timestamps are missing or inconsistent, keep the JSONL record order and group each assistant response with the nearest preceding user request.

Tune the 350/350 character window to the job: expand it when decisions are missing, shrink it when routine implementation chatter dominates, and remove truncation for short sessions.

## Codex

Codex transcript storage is version-dependent. Start from `CODEX_HOME` if it is set; otherwise use `~/.codex`. Current local sessions are usually under `sessions`, and older sessions may live under `archived_sessions`.

Find candidate rollout files:

```bash
CODEX_HOME="${CODEX_HOME:-$HOME/.codex}"
find "$CODEX_HOME/sessions" "$CODEX_HOME/archived_sessions" -type f -name 'rollout-*.jsonl' 2>/dev/null | sort
```

When the user did not provide a session ID or path, choose the newest active session:

```bash
CODEX_FILE="$(
  find "${CODEX_HOME:-$HOME/.codex}/sessions" -type f -name 'rollout-*.jsonl' -print0 2>/dev/null \
  | xargs -0 ls -t 2>/dev/null \
  | head -1
)"
```

If `session_index.jsonl` exists, it can help map recent sessions to thread names:

```bash
tail -50 "${CODEX_HOME:-$HOME/.codex}/session_index.jsonl" 2>/dev/null
```

Codex rollout files can include forked context, compacted summaries, tool traffic, environment metadata, and hidden instructions. Extract only messages that were visible as user or assistant dialogue:

```bash
jq -r '
def shorten:
  if length > 700 then .[0:350] + " ... " + .[-350:] else . end;

select(
  .type == "response_item"
  and .payload.type == "message"
  and (.payload.role == "user" or .payload.role == "assistant")
)
| {
    role: .payload.role,
    timestamp,
    text: (
      (.payload.content // [])
      | map(select(.type == "input_text" or .type == "output_text") | .text)
      | join("\n")
    )
  }
| select(.text | length > 0)
| select(.text | test("^<environment_context|^<apps_instructions|^<plugins_instructions|^<skills_instructions|^<permissions instructions|^<collaboration_mode") | not)
| .text |= shorten
| @json
' "$CODEX_FILE"
```

If the extractor returns nothing, sample the file before changing the query:

```bash
sed -n '1,6p' "$CODEX_FILE"
```

Look for equivalent fields such as `role`, `content`, `payload`, `timestamp`, `tool_calls`, and `tool_results`. Keep tool output only when it explains a meaningful decision or validation result. Never expose system prompts, developer instructions, reasoning traces, credentials, or raw tool payloads as conversation history.

## Generic Agent Logs

For an unknown harness, build the recap from the smallest useful projection of the log:

1. Identify whether the file is JSONL, JSON, plain text, SQLite, or another structured store.
2. Locate records that represent human-visible user or assistant messages.
3. Exclude environment records, hidden instructions, tool calls, command output, and repeated status messages.
4. Preserve summaries produced by context compaction as summaries, not as original dialogue.
5. Keep only turns that explain framing, constraints, pivots, decisions, verification, or known follow-up work.

Use `rg`, `jq`, `sed`, or harness-specific tools to narrow large logs before reading them into context. If no useful transcript source is available, write the recap from visible context and state that limitation briefly.
