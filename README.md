# noli-models

Public, data-only model catalogs for [Noli](https://github.com/MelonSmasher/noli).

## `claude-models.json`

Noli's built-in Claude model catalog. Noli never lists Claude models from Anthropic (Claude Code 2.1.309 does not either). It embeds a byte-identical copy of this file as its offline floor. It refreshes from

    https://raw.githubusercontent.com/MelonSmasher/noli-models/main/claude-models.json

How Noli refreshes it:

- **When:** at startup and every 24 hours, with `If-None-Match`.
- **What it accepts:** at most 1 MiB, strictly validated against `claude-models.schema.json`.
- **Versioning:** a copy is installed only when its `catalog_version` is greater than the one in effect. The last accepted copy persists locally. Precedence is fetched, then persisted, then embedded.

The file is data only. It can add or retire models and change limits, capability facts and prices. It cannot change endpoints, headers, betas, credentials or request shaping beyond the capability facts Noli already reads.

### Publishing

1. Edit `claude-models.json` and increase `catalog_version`. A file whose version is not greater than the one Noli holds is ignored.
2. Validate it against `claude-models.schema.json`, for example with `check-jsonschema --schemafile claude-models.schema.json claude-models.json`.
3. Commit to `main`.

Noli's own build embeds whatever copy it was released with. Keep the schema (`schema_version` 1) backward compatible. A breaking change needs a new file name.

### Sources (catalog_version 1)

**Model set, families and defaults** come from Claude Code 2.1.309's baked catalog `C4r`:

- Binary `claude-2.1.309-linux-x64`, sha256 `4d704a8aecbe038affa3000f498ae17c1b4f753add8facb8764120affc99b658`.
- Location: `chunk-eqkryfm7.js`, orig offset 240723103.
- Data taken: the 22 models, their families, display names and first-party ids (kept as `aliases`).
- Context windows (`context.window`/`maximum_window`) and output limits (`max_output_tokens.default` → `default_max_tokens`, `.upper` → `max_output_tokens`).
- Pricing tiers, resolved to numbers.
- `best`, `latest_per_family` and the alias defaults (`families`).

**Capability facts** come from the first-party resolution of Claude Code's lookups, with these sources per fact:

| Fact | Claude Code source |
|---|---|
| `thinking.supported` | `lZs` |
| `thinking.adaptive` | `EKo` |
| `thinking.disabled` | `!LLe` (rejects disabled thinking) |
| `thinking.interleaved` | `yer` |
| `effort` levels | `hw` / `rce` / `jbt` |
| `sampling` | `G2r` (accepts temperature) |
| `context_management` | `W0` |
| `conversation.tool_changes` | catalog `mid_conv_tool_change` |
| `conversation.effort_changes` | catalog `per_turn_effort` |

**Effort defaults** come from the published catalog seed (version 2844, `cc.model_selector_state.thinking_by_model`), then from the catalog's `default_effort`.

**Main and small-fast defaults:**

- `main`: `bE()` gives `claude-opus-5-5` for API keys and most subscriptions.
- `small_fast`: `ub()`/`GL()` gives `claude-haiku-5-5`, used for titles.

**`verify`** is the model id for the API-key check, `MLe()`. It is hard-wired to `HS().haiku45`, which is `claude-haiku-4-5-20251001`.

**Lifecycle** (`status`, `retirement`, `replacement`) comes from <https://platform.claude.com/docs/en/about-claude/model-deprecations>, as of 2026-10-28. Claude Code's catalog carries no lifecycle data.

**Noli-observed facts**, not present in `C4r`:

- `thinking.display`: Noli's display gate. It is off for Claude 3.x and Haiku 4.5.
- `thinking.budget_tokens`: on for every thinking model except Haiku 5.5.
- `conversation.per_turn_control` / `system_clear_at`: observed for Haiku 5.5 only. This is a remote capability table in Claude Code.
- `structured_outputs` / `strict_tools`: from Anthropic's published supported-model list.
- `inputs`.

**`pricing_only`:** `claude-mythos-preview` is deprecated and has no Claude Code catalog entry. It is kept as verified flat pricing for usage accounting only. Its rates come from Noli's pricing feed.
