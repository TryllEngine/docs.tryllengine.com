# Filter grammar

A shared JSON predicate grammar compiled by the server into a typed `CompiledFilter` and evaluated with no allocation on the hot path. Two nodes consume it today, against two different operand sets:

- **[`Retrieve`](https://docs.tryllengine.com/v0.8/reference/nodes/retrieve.html)`.filter`** — applied against each candidate record's metadata during vector search. Validated against the knowledge base's [metadata schema](https://docs.tryllengine.com/v0.8/reference/embedded-string-storage.html#metadata-schema) at compile time — unknown fields, wrong types, and malformed grammar are rejected before the filter takes effect. Has no slot view: a `{"slot":…}` operand is rejected with `filter.slot_unavailable`.
- **[`Branch`](https://docs.tryllengine.com/v0.8/reference/nodes/branch.html)`.condition`** (under `test = Expression`) — has no knowledge base (a `{"knowledge":…}` operand is rejected with `filter.knowledge_unavailable`) but *does* bind a slot view, so `{"slot":…}` operands work.

The grammar itself — ops, operand shapes, validation rules — is identical for both; the sections below note where a node's own capabilities narrow it.

## Top-level node shape

Every filter node is a JSON object with an `op` key. The supported ops:

| `op`                              | Shape                                       | Meaning                                                                   |
| --------------------------------- | ------------------------------------------- | ------------------------------------------------------------------------- |
| `"and"`                           | `{"op":"and","args":[...]}`                 | All children must evaluate to true.                                       |
| `"or"`                            | `{"op":"or","args":[...]}`                  | At least one child must evaluate to true.                                 |
| `"not"`                           | `{"op":"not","arg":{...}}`                  | Invert the child.                                                         |
| `"eq"` / `"ne"`                   | `{"op":"eq","lhs":<op>,"rhs":<op>}`         | Equality / inequality.                                                    |
| `"lt"` / `"le"` / `"gt"` / `"ge"` | same shape as `eq`                          | Ordering; numeric only.                                                   |
| `"in"`                            | `{"op":"in","needle":<op>,"haystack":<op>}` | Membership; `haystack` must be a `set<string>` field or an array literal. |

## Operands

An operand is one of:

```
{ "knowledge": "<field-name>" }     // reference a knowledge-side field (Retrieve only)
{ "var":       "<variable-name>" }  // reference a declared per-agent Variable
{ "value":     <json-literal> }     // scalar literal (int | float | string | bool)
                                    //   or string-array literal (only as `in.haystack`)
{ "slot":      "<slot-name>" }      // reference a turn-local slot (Branch only)
```

`{"var": "<name>"}` references a variable declared in this agent's [`CreateAgentRequest.variables`](https://docs.tryllengine.com/v0.8/concepts/agent-variables.html) — the type is fixed at declaration and checked against the other operand exactly like a `{"knowledge": ...}` field (including `int`↔`float` promotion). Unlike a `{"value": ...}` literal, a `var` operand is resolved to the variable's **current** value on every evaluation — mutate it with [`Variables().SetXxx`](https://docs.tryllengine.com/v0.8/concepts/agent-variables.html) and the *next* retrieval or branch decision reflects the change, with no `ChangeParam` round-trip and no filter recompile. An unknown variable name fails compilation with `filter.unknown_variable`. The `agent` namespace is reserved but not implemented — supplying `{"agent": "..."}` returns `filter.unknown_namespace` with a hint to use `var` instead.

### Slot operand

`{"slot": "<name>"}` references a [slot](https://docs.tryllengine.com/v0.8/concepts/slots-and-inter-node-value-passing.html) written earlier this turn — the same names a `Generate`/`Transform`/`CannedResponse` node writes and an `input` param resolves. It is only meaningful where a node actually binds a slot view; today that is [`Branch`](https://docs.tryllengine.com/v0.8/reference/nodes/branch.html)`.condition` alone. A few properties fall straight out of what a slot *is*:

- **Always string-typed.** Slots hold rendered/generated text, so a `{"slot":…}` operand never satisfies `lt`/`le`/`gt`/`ge` — those require a numeric type on both sides, and there is no automatic string→number coercion (the door left open for a future `{"slot":"n","as":"int"}` form).
- **Not valid as an `in` haystack.** A slot holds one text value; `in` over a slot has no non-arbitrary meaning. Put it in `needle` instead — attempting `haystack: {"slot":…}` fails compilation naming the fix. Watch the inverse trap too: a `Transform`-rendered `set<string>` variable is comma-joined into one string, so it *reads* like a set when viewed through `{"slot":…}` but is really one opaque string — `Contains`/`Equals` will compare against the whole joined text, not one element.
- **Resolved once per node execution**, not per record (there is no record loop for `Branch`) — the resolved binding is looked up from the interned slot table `CompiledFilter::GetReferencedSlots()` built at compile time.
- **Interned and validated by the graph compiler, not by name at evaluation time.** A slot reference resolves to a dense index at compile time; a slot name that can never be available on any path into the node fails agent creation the same way any other never-available slot read does — see [Slots and inter-node value passing](https://docs.tryllengine.com/v0.8/concepts/slots-and-inter-node-value-passing.html).
- **Missing ≠ present-but-blank.** See [Missing-value semantics](#missing-value-semantics) below — a slot the graph never wrote on this path is `Missing`; a slot written with empty text is present.

## Which operands each parameter accepts

Not every operand kind is legal in every position, and the legal set also depends on which node is doing the compiling (a `{"knowledge":…}` operand only makes sense where there is a knowledge base; a `{"slot":…}` operand only where a slot view is bound):

| Position                                    | `knowledge`                      | `var`               | `value`         | `slot`                                           |
| ------------------------------------------- | -------------------------------- | ------------------- | --------------- | ------------------------------------------------ |
| `eq`/`ne`/`lt`/`le`/`gt`/`ge` — `lhs`/`rhs` | ✓ (Retrieve)                     | ✓                   | ✓ scalar        | ✓ (Branch) — never satisfies `lt`/`le`/`gt`/`ge` |
| `in` — `needle`                             | ✓ (Retrieve)                     | ✓                   | ✓ scalar        | ✓ (Branch)                                       |
| `in` — `haystack`                           | ✓ `set<string>` field (Retrieve) | ✓ `set<string>` var | ✓ array literal | ✗ rejected — see [Slot operand](#slot-operand)   |

`{"knowledge":…}` inside a `Branch.condition` fails with `filter.knowledge_unavailable` (Branch declares an empty `KnowledgeBaseConfig{}` by design — it needs no KB schema). `{"slot":…}` inside a `Retrieve.filter` fails with `filter.slot_unavailable` (`Retrieve` binds no slot view, so a slot ref would otherwise silently evaluate `Missing` on every record, forever).

## The five most common patterns

```
// 1. var.level >= knowledge.min_level
//    Declared: variables = {"level": 12}. Mutate with Variables().SetInt("level", 13).
{ "op": "ge",
  "lhs": { "var": "level" },
  "rhs": { "knowledge": "min_level" } }

// 2. var.character_class == knowledge.character_class
{ "op": "eq",
  "lhs": { "var": "character_class" },
  "rhs": { "knowledge": "character_class" } }

// 3. var.character_class IN knowledge.character_classes
{ "op": "in",
  "needle":   { "var":       "character_class" },
  "haystack": { "knowledge": "character_classes" } }

// 4. knowledge.quest_reached IN var.quests_reached
//    Declared: variables = {"quests_reached": []} (set<string>).
//    Mutate with Variables().AddToSet("quests_reached", "lost_amulet").
{ "op": "in",
  "needle":   { "knowledge": "quest_reached" },
  "haystack": { "var":       "quests_reached" } }

// 5. one-off literal, still supported when a value never changes at runtime
{ "op": "eq",
  "lhs": { "value": "warrior" },
  "rhs": { "knowledge": "character_class" } }
```

Composing the gates:

```
{ "op": "and", "args": [
    { "op": "ge",
      "lhs": { "value": 12 },
      "rhs": { "knowledge": "min_level" } },
    { "op": "in",
      "needle":   { "value":     "mage" },
      "haystack": { "knowledge": "character_classes" } }
] }
```

## Validation rules

A filter fails to compile if any of the following holds — this list is identical for `Retrieve.filter` and `Branch.condition`; only where the error surfaces differs (next section):

- An `op` value is not in the supported set.
- An operand uses the reserved `agent` namespace.
- A `{"var": "..."}` reference names a variable not declared in `CreateAgentRequest.variables` (`filter.unknown_variable`).
- A `{"knowledge": "..."}` reference names a field absent from the schema, or a field declared `filterable: false` — or appears at all where no knowledge base is bound (`filter.knowledge_unavailable`).
- A `{"slot": "..."}` reference appears where no slot view is bound (`filter.slot_unavailable`), or as an `in.haystack`.
- Operand types are incompatible:
- `lt`/`le`/`gt`/`ge` on non-numeric operands (a slot operand always fails this check).
- `in` whose `haystack` is neither a `set<string>` field/variable nor an array literal.
- `in` whose `needle` type does not match the haystack's element type.
- `eq`/`ne` on mismatched types (with `int`↔`float` promotion as the only permitted exception).
- An array literal appears anywhere other than as the `haystack` of an `in`.
- `and`/`or` `args` array is empty.
- Tree depth exceeds 16 or total node count exceeds 256.

Errors include the JSON path of the offending node — e.g. `filter.type_mismatch: at args[1].rhs: cannot compare string with int`.

## Where a compile failure surfaces

*When* the filter is compiled, and *which node* owns it, determine what a failure does — these are genuinely different, not a single uniform `3007`:

**At agent creation.** Compiling `Retrieve.filter` or `Branch.condition` as part of `CreateAgent` fails the **whole agent** with [`3003 GraphCompilationFailed`](https://docs.tryllengine.com/v0.8/reference/error-codes.html#3xxx-agent) — bad JSON in either field is caught before the agent exists.

**At runtime, via `ChangeAgentParam` — the two nodes diverge:**

| Node                                        | Behaviour on an invalid mutation                                                                                                                                                                                                                                                                                                                                                                        |
| ------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `Retrieve.filter` / `ClassifyIntent.filter` | **Silently no-ops.** The validator has no KB schema to check against at this layer, so it returns `Ok`; `OnFilterChanged` logs a `warn` and keeps the previous filter in effect. The server still `Ack`s the request. This is a known gap, not the intended design — don't rely on it to catch a typo in `filter` JSON sent from a game.                                                                |
| `Branch.condition`                          | **Rejected with [`3007 InvalidParamValue`](https://docs.tryllengine.com/v0.8/reference/error-codes.html#3xxx-agent)**, carrying the JSON path and message (`FilterError::Format()`), and the node's active condition is left untouched. `ValidateBranchMutationParams` fully re-compiles the candidate condition — Branch has no KB schema to lack an excuse with, so there is no silent-Ack path here. |

Branch's mutation validator is the deliberately stricter of the two — see `docs/research/workflow/conditional-routing-if-and-switch-nodes.md`. The `Retrieve`/ `ClassifyIntent` silent-Ack behaviour is filed as a follow-up to align with it.

## Missing-value semantics

**Missing knowledge field.** When the schema declares a field as `optional: true` and a record omits the field, every comparison or `in` operation referencing that field on that record evaluates to **false**. This is intentional — to mean "no restriction applies" the filter author should encode that explicitly with `or` and a separate gating field. Retrieve / ClassifyIntent still see only a boolean (`Test()`): `true` keeps a candidate, anything else rejects it.

**Missing slot → `Undefined`.** A `{"slot":…}` operand is `Undefined` when the graph never wrote that slot on the path taken this turn (not when it was written blank — see [Branch's exit routes](https://docs.tryllengine.com/v0.8/reference/nodes/branch.html#exit-routes)). Comparisons and `in` involving an `Undefined` operand evaluate to **`Undefined`**, and `and` / `or` / `not` use Kleene three-valued logic:

| Expression         | Slot missing              | Present, equal | Present, unequal |
| ------------------ | ------------------------- | -------------- | ---------------- |
| `eq(a, b)`         | `Undefined`               | `true`         | `false`          |
| `ne(a, b)`         | `Undefined`               | `false`        | `true`           |
| `not(eq(a, b))`    | `Undefined`               | `false`        | `true`           |
| `and(…, false, …)` | `false` (false dominates) | —              | —                |
| `or(…, true, …)`   | `true` (true dominates)   | —              | —                |

Branch maps `true` → `then`, and both `false` and `Undefined` → `else`, logging a **warning** and setting `operand_missing` when the top-level result is `Undefined`.

Prefer a positive allowlist: `in(slot, allowed)` with `then` = allowed / `else` = blocked. A missing slot then fails closed to blocked. Do **not** rely on `not(in(slot, var))` or `not(eq(...))` to treat "no value" as a match — those stay `Undefined` and take `else`.

## Mutating the filter at runtime

`Retrieve.filter` is a mutable parameter:

```
agent.change_param("retrieve", "filter", json.dumps({
    "op": "ge",
    "lhs": {"value": player.level},
    "rhs": {"knowledge": "min_level"},
}))
```

The previous filter remains in effect until the new JSON compiles successfully — see [Where a compile failure surfaces](#where-a-compile-failure-surfaces) above for why an invalid update is a silent no-op here but a hard `3007` on `Branch.condition`. Sending an empty string clears the filter.

## Related

- [Retrieve node](https://docs.tryllengine.com/v0.8/reference/nodes/retrieve.html)
- [Branch node](https://docs.tryllengine.com/v0.8/reference/nodes/branch.html) — the other consumer of this grammar, via `condition`
- [Embedded String Storage](https://docs.tryllengine.com/v0.8/reference/embedded-string-storage.html)
- [Agent Variables](https://docs.tryllengine.com/v0.8/concepts/agent-variables.html)
- [Slots and inter-node value passing](https://docs.tryllengine.com/v0.8/concepts/slots-and-inter-node-value-passing.html)
- [Change Agent Parameters at Runtime](https://docs.tryllengine.com/v0.8/how-to/change-agent-parameters.html)
