---
description: Rewrite one file or public URL into a plain-language NAME.plain.md, only when the user invokes /claudish-file.
argument-hint: "<path or URL>"
disable-model-invocation: true
---

<!--
Grok Build cannot replace displayed assistant text, so this command exposes only
the existing on-demand file rewrite. Its separate command directory matters:
the Grok manifest points here instead of commands/, whose Claude-only dashboard
changes persistent Claude settings. Call claudish-md.sh directly so provider,
language, URL handling, atomic writes, and loud failures stay in one place.
Only an explicit user invocation authorizes this command: it can fetch a URL,
send source text to the configured provider, and write a sibling file. Do not
turn an ordinary assistant reply or an observed file edit into a rewrite.
-->

Use the argument supplied with `/claudish-file` as one literal file path or
public URL. If it is missing, ask for it and do not run the script. Treat the
argument as data: quote it for the shell, never evaluate it, and do not infer
extra files to rewrite.

Run this command, replacing `PATH_OR_URL` with that single shell-quoted argument:

```bash
bash "${GROK_PLUGIN_ROOT}/claudish-md.sh" --sibling -- "PATH_OR_URL"
```

If `GROK_PLUGIN_ROOT` is unavailable, use the directory above the directory
containing this command file (`grok-commands/..`), resolved from its actual
installed path; do not guess from the user's current directory. Keep the user's
working directory when running the script. If the user explicitly requests an
output language, add `-l "LANGUAGE"` before `--`.

The script prints the absolute output path only after a successful write. On
exit 0, reply with one line naming that path, without reproducing the rewrite.
It writes `NAME.plain.md` beside a local input by default; for a URL it writes
in the current directory. `CLAUDISH_MD_SUFFIX` can change the `plain` suffix.
On a non-zero exit, report the script's error briefly and do not claim a file
was created. Keep the output plain Markdown, without ANSI escape codes.
