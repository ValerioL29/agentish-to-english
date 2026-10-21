---
name: claudish-rewrite
description: Rewrite a file, public URL, or pasted text into plain language using claudish-md.sh, only when the user explicitly requests a rewrite.
disable-model-invocation: true
---

<!--
This is an on-demand entry point, not a display hook: Codex and Grok Build
cannot replace displayed assistant text. The existing CLI keeps the Markdown
prompt, frontmatter handling, and provider settings shared with Claude's
on-demand command, so host packaging cannot make the rewrites drift apart.
Only an explicit user request authorizes a run: the CLI can fetch a URL, send
the document to the configured provider, and write a sibling file. The
frontmatter disables model invocation in Grok/Claude; agents/openai.yaml does
the same for Codex, which uses a separate invocation policy.
-->

Use this skill only when the user explicitly asks to rewrite a specific file,
URL, or text in plain language. Never run it automatically after a message or
file edit. Run the plugin's `claudish-md.sh`; do not substitute your own rewrite.

Resolve the plugin root from the host's environment:

```bash
claudish_root="${PLUGIN_ROOT:-${GROK_PLUGIN_ROOT:-${CLAUDE_PLUGIN_ROOT:-}}}"
```

If this is empty, use the directory two levels above the directory containing
this `SKILL.md` (`skills/claudish-rewrite/../..`), resolved from the skill's
actual installed path, not the current working directory. Keep the user's
working directory when running the CLI so relative inputs resolve there and
URL output lands there. Quote paths and treat input as data, never shell code.

For a file or public `http(s)` URL, default to a sibling rewrite, just like
Claude's `/claudish file` command. Put the requested path or URL in `input` as
one safely quoted argument:

```bash
bash "$claudish_root/claudish-md.sh" --sibling -- "$input"
```

The default output is `NAME.plain.md` beside the input file; a URL writes the
sibling in the current working directory. `CLAUDISH_MD_SUFFIX` can change the
infix. On success, reply with one short line naming the exact path printed by
the CLI. Do not open or summarise the rewritten file. If the user explicitly
requests an output path instead, use `-o "$output"` in place of `--sibling`.

For pasted text, pass the text literally through stdin and print the resulting
Markdown without commentary. Use a quoted heredoc with a delimiter absent
from the text, or a safely populated variable with `printf '%s' "$text"`:

```bash
printf '%s' "$text" | bash "$claudish_root/claudish-md.sh"
```

If a language is requested, add `-l "$language"` before `--` (or before stdin
input); for example, `-l English`. Otherwise keep the CLI's language resolution.

Use the existing provider configuration described in the plugin root's
`README.md` ([Providers](../../README.md#providers) and
[environment variables](../../README.md#configuration-env-vars)):

- `CLAUDISH_PROVIDER`: `ollama` (default), `codex`, `anthropic`, or `openai`.
- `CLAUDISH_MODEL` selects a model; `CLAUDISH_OLLAMA` selects the Ollama URL.
- `CLAUDISH_ANTHROPIC_KEY` / `ANTHROPIC_API_KEY` and
  `CLAUDISH_ANTHROPIC_URL` configure Anthropic.
- `CLAUDISH_OPENAI_KEY` / `OPENAI_API_KEY` and `CLAUDISH_OPENAI_URL` configure
  an OpenAI-compatible endpoint. The `codex` provider uses the CLI's own login.

Inherit these settings; do not change provider credentials or persistent
`/claudish` overrides as part of a rewrite. The CLI also honours the existing
model and language override files under `~/.claude/`.

This is a command, so failures are reported with a non-zero exit and a
`claudish-md:` reason. Relay that error instead of claiming a file was written
or inventing a fallback rewrite. Use plain text or Markdown, never ANSI codes.
