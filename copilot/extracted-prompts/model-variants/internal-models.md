# Internal / Hashed Model Variants

These variants serve models identified by obfuscated (SHA-256 hashed) family
names or internal model identifiers. They share the same structural template
as the generic fallback prompt with minor differences.

---

## Registration Summary

| Registration | Family Prefixes | Matcher | Prompt Class | Reminder | Notes |
|---|---|---|---|---|---|
| `uBt` | (none) | `n_e(e)` → Vwi (1 hash) | `lBt` (generic fallback) | `kI` | Exact same as generic |
| `fBt` | (none) | `a_e(e)` → `family.includes("minimax")` | `ABt` | `kI` | Minimax models |
| `vBt` | `vscModelA` | `r_e(e)` → i_n (2 hashes) | `gBt` | `n$e` | Internal model A |
| `_Bt` | `vscModelB` | `i_e(e)` → o_n (2 hashes) | `hBt` | `xBt` | Internal model B |
| `wBt` | `vscModelC` | `l_n(e)` → hash list | `bBt` | `EBt` | Internal model C |
| `CBt` | `vscModelD` | `u_n(e)` → hash list | `yBt` | `n$e` | Internal model D |

All use the standard `uo` (verbose) identity rules and `Vr` (standard) safety rules,
except where noted.

---

## ABt — Minimax Prompt

Nearly identical to the generic fallback (`lBt`) but uses a `<role>` section
wrapper instead of starting with `<parallel_tool_use_instructions>`:

<role>
You are an expert AI programming assistant, working with a user in the VS Code editor.

When asked for your name, you must respond with "GitHub Copilot". When asked about the model you are using, you must state that you are using GitHub Copilot.

Follow the user's requirements carefully & to the letter.

Follow Microsoft content policies.
Avoid content that violates copyrights.
If you are asked to generate content that is harmful, hateful, racist, sexist, lewd, or violent, only respond with "Sorry, I can't assist with that."

Keep your answers short and impersonal.
</role>

Then continues with the same sections as generic fallback:
`parallel_tool_use_instructions` → `instructions` → `toolUseInstructions` → `editFileInstructions` → `outputFormatting`

**Key difference**: Identity and safety rules are inlined in the `<role>` section
rather than being injected as separate components.

---

## gBt — vscModelA Prompt

Structurally identical to generic fallback (`lBt`).
Starts with `parallel_tool_use_instructions` using the verbose format
(same as generic-fallback.md).

---

## hBt — vscModelB Prompt

Structurally identical to generic fallback (`lBt`).
Starts with `parallel_tool_use_instructions` using the verbose format.

---

## bBt — vscModelC Prompt

Structurally identical to generic fallback (`lBt`).
Starts with `parallel_tool_use_instructions` using the verbose format.

---

## yBt — vscModelD Prompt

Slightly different from the others:

Starts with plain text (no section wrapper):
"You are an expert AI programming assistant, working with a user in the VS Code editor."

Then uses `parallel_tool_use_instructions` with a different opening:
"The `multi_tool_use` wrapper may not be available in every environment. If it is available, follow the parallel tool use instructions..."

This is different from the standard "Using `multi_tool_use` to call multiple tools in parallel is ENCOURAGED..." used by the others.

---

## Hash Lists (Obfuscated Family Names)

These SHA-256 hashed strings represent model family names that can't be
recovered without the original strings:

| Variable | Count | Used By |
|---|---|---|
| `Vwi` | 1 hash | `n_e(e)` → `uBt` → generic fallback |
| `Wwi` | 2 hashes | `B9(e)` → `Zxe` → Gemini |
| `i_n` | 2 hashes | `r_e(e)` → `vBt` → vscModelA |
| `o_n` | 2 hashes | `i_e(e)` → `_Bt` → vscModelB |
| `a_n` | 1 hash | Part of `c_n` union |
| `s_n` | unknown | Part of `c_n` union |
| `c_n` | union of i_n + o_n + a_n + s_n | `C_n(e)` → `DBt` → GPT-5.x internal |
| `Gwi` | 1 hash | `g3e(e)` → `XBt` → unknown model |
| `Zwi` | 4 hashes | `Cee(e)` → `Kxe` → GPT-5.4 targets |
| `Hwi` | 1 hash | Used in other matchers |

---

## Summary

These variants exist to handle models from different providers with
slightly customized prompts. In practice, most render near-identical
content to the generic fallback. The primary differences are:

1. **ABt (Minimax)**: Inlines identity/safety in `<role>` section
2. **yBt (vscModelD)**: Different parallel tool use wording
3. **gBt, hBt, bBt**: Functionally identical to generic fallback
4. **lBt (via uBt)**: IS the generic fallback
