# Filter LLM Output Artifacts

Strip non-spoken roleplay markup from streamed `Generate` / `GenerateAndSpeak` output before it reaches the client, dialogue history, or TTS: speaker-name prefixes (`Barnaby: …`), asterisk emotes (`*sighs*`), and parenthetical / bracket stage directions.

All flags default **off**. Filtering is opt-in per node. Transformed text is authoritative for the slot, wire answer, history, and TTS.

Prerequisites

- A `Generate` or `GenerateAndSpeak` node.
- Willingness to mutate history: filtered text is what the agent remembers saying on later turns.

______________________________________________________________________

## 1. Enable the filter on the node

Set fields under `output_filter` (`OutputFilterOverrides`):

| Field                  | Default | Purpose                                              |
| ---------------------- | ------- | ---------------------------------------------------- |
| `strip_speaker_prefix` | false   | Remove a leading `Name:` tag once per turn           |
| `max_prefix_bytes`     | 16      | UTF-8 byte budget for the prefix scan (`[1, 64]`)    |
| `strip_asterisk_spans` | false   | Delete `*…*` spans (first-close wins)                |
| `strip_paren_spans`    | false   | Delete `(…)` spans                                   |
| `strip_bracket_spans`  | false   | Delete `[…]` spans                                   |
| `max_span_bytes`       | 64      | UTF-8 byte budget while matching a span (`[1, 256]`) |

```
from tryll_client.graph import GenerateParams, OutputFilterOverrides, SendAnswer

GenerateParams(
    send=SendAnswer.Streamed,
    output_filter=OutputFilterOverrides(
        strip_speaker_prefix=True,
        strip_asterisk_spans=True,
        strip_paren_spans=True,
    ),
)
```

On the Generate / GenerateAndSpeak node params, expand **Output Filter** and enable the flags you need. Budgets are clamped in the inspector.

______________________________________________________________________

## 2. What the rules do

- **Speaker prefix (R1).** At the first non-whitespace byte of the turn, if a `:` followed by whitespace appears within the budget, and the candidate starts with `A–Z`, has at most three ASCII words, and contains no `.` `!` `?` `,`, the tag is deleted. ASCII-only. Residual false positives such as `One rule: …` are possible — keep the flag off unless you want this cleanup.
- **Paired spans (R2).** Enabled delimiter pairs delete open…close including content. First close wins; nested opens are treated as content. Unmatched or over-budget candidates are emitted unchanged (benign failure).
- **Whitespace.** Leading/trailing ASCII whitespace is trimmed; a space on both sides of a deleted span squeezes to one space when still stream-safe.

Filtering runs **before** `substitute_agent_variables`. Markers inside a deleted span are never substituted; values introduced by substitution are never re-filtered.

______________________________________________________________________

## 3. Diagnostics

When `enable_diagnostics` is on, each Generate / GenerateAndSpeak node entry may include `output_filter_removals`:

```
{
  "rule": "strip_speaker_prefix",
  "matched": "Barnaby: ",
  "source_byte_offset": 0,
  "output_byte_offset": 0
}
```

At most 256 records are kept (`output_filter_removals_truncated` when exceeded). Telemetry omits matched content when user-content telemetry is off.

______________________________________________________________________

## When not to use this

- Leave flags off for free-form chat that may legitimately start with `Label: value` lines.
- Do not expect curly quotes, Unicode names, or third-person narration without markup to be cleaned — those are prompt/eval concerns or future flags.
- Quote wrapping (`"…"` around a whole turn) is not stripped in this release.
- Tool-call markup (`<|tool_call>…`, `[TOOL_CALLS]`) is not an artifact to filter. It means a node answered with a tool call. See [Fix tool-call text in answers](https://docs.tryllengine.com/v0.8/how-to/fix-tool-call-text-in-answers.html).

See also [Substitute Agent Variables in LLM Output](https://docs.tryllengine.com/v0.8/how-to/substitute-agent-variables-in-output.html) for the sibling stream transform.
