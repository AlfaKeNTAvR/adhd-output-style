# ADHD output style for Claude Code

An **output style** that reshapes every Claude Code response: result first, state
restated each turn, concerns deferred but never dropped, exactly one next action
at the end.

Derived from [**ayghri/i-have-adhd**](https://github.com/ayghri/i-have-adhd) by
Ayoub Ghriss (MIT), which ships the same idea as a *skill*. This is a port to
Claude Code's output-style mechanism, plus changes for agentic (not chat) use.
See [Differences](#differences-from-i-have-adhd).

## Install

Paste into Claude Code:

```text
Install the ADHD output style from https://github.com/AlfaKeNTAvR/adhd-output-style, refer to the repo's AGENTS.md for instructions.
```

Or by hand, in three steps:

1. Save [`output-styles/adhd.md`](output-styles/adhd.md) to `~/.claude/output-styles/adhd.md`.
2. Add `"outputStyle": "ADHD"` to `~/.claude/settings.json` for every project, or
   run `/config` -> **Output style** -> **ADHD** to set it for the current project only.
3. Run `/clear`, or start a new session. The style is read at session start, so an
   edit never applies to the session that made it.

Full instructions, including scope and precedence between settings files and the
optional `CLAUDE.md` companion rules: [INSTALL.md](INSTALL.md).

## Why an output style and not a skill

This is the whole reason the port exists.

A **skill** is a document the model chooses to load. Even with
`disable-model-invocation` and an explicit `/command`, staying in the style is
something the model has to keep deciding to do, which is why the upstream skill
needs a "Persistence" section telling Claude the rules have not expired. Over a
long session, that decision drifts.

An **output style** is not a document the model reads. It modifies Claude Code's
system prompt, and the harness keeps reinforcing it. From the official docs:

> Output styles directly modify Claude Code's system prompt.

> When you select a style other than Default, Claude Code also reminds Claude of
> the style during the conversation.

That second sentence is the mechanism. The reminder is injected by the harness on
an ongoing basis, not written by the model and not dependent on the model
remembering.

| | Skill | `CLAUDE.md` | Output style |
|---|---|---|---|
| How it reaches Claude | Loaded when invoked or judged relevant | User message after the system prompt | Modifies the system prompt |
| Stays active | Model has to keep applying it | Loaded once per session | Harness re-reminds during the conversation |
| Turning it on | `/i-have-adhd` each session | Always | `/config` once, persists in settings |

Source: [Claude Code output styles](https://code.claude.com/docs/en/output-styles).

## Differences from i-have-adhd

Same core: no preamble, no recap, no closers, capped lists, matter-of-fact
errors, one next action, state restated every turn. What changed:

| | i-have-adhd | This style |
|---|---|---|
| **Mechanism** | Skill, invoked with `/i-have-adhd`, kept alive by a "Persistence" section | Output style, selected in `/config`, reinforced by the harness |
| **First line** | Lead with the *next action* | Lead with the *result*: what happened, or the answer. In an agent harness the work is usually already done, so the outcome is the news; the next action moves to the end |
| **Next action** | Rule 3, end with one concrete step | Same, plus a hard ordering rule: the `Next:` line is the literal last line, nothing after it |
| **Estimates** | Specific clock time ("about 15 minutes") | Blast radius: files touched, migrations, whether the API surface changes. Clock time only for work *you* must do (sudo, hardware, interactive logins). An agent's wall-clock is not yours |
| **Non-blocking concerns** | Tangents suppressed or offered as a follow-up question | A named home: `## Worth knowing`, one line each, ranked. Deferred, never dropped. Anything blocking, unsafe, or needing approval is forced inline and up front instead |
| **Numbered steps** | Rule 2, number multi-step tasks in prose | Use the task tool for multi-step work and let the checklist do the restating, rather than narrating the plan as prose |
| **Register** | Not specified | Dropped-grammar shorthand: fragments, no articles, in chat and code comments. Never drop a unit, a threshold, or a reason. Doc files and commit text stay grammatical |
| **Banned surface** | Idioms removed in the pre-send check | Also no emojis, no em dashes or en dashes, English only |
| **Correctness floor** | "When to break the rules" list | Stated as a rule: error reports, failing test output, security warnings, and destructive-action confirmations keep their full content. Brevity never overrides correctness |
| **Precedence** | Harness system prompt outranks the skill | The style governs *shape* only. On anything else, your `CLAUDE.md` wins |

The upstream "when to break the rules" cases (explain-fully requests, destructive
actions, real ambiguity) are kept, folded into the rules they modify.

## Requirements

Claude Code with output-style support. Nothing else: no dependencies, no hooks,
no scripts.

Tested on Claude Code 2.1.252, Opus 5, high effort level. It should hold on any
model, since it is prompt text and nothing else, but that is the configuration it
was written against and shaped by. Reports from other models and effort levels are
welcome.

Note: `/output-style` as a standalone command was deprecated in 2.1.73 and removed
in 2.1.91. On current versions the selector lives in `/config`.

## Limitations

- **Main conversation only.** Subagents run their own system prompt, so the style
  does not change how they respond.

## Customizing

- **Rename**: change `name:` in the frontmatter. That string is what `/config` shows.
- **Turn a rule off**: delete its numbered item. Rules are independent; none
  cross-references another by number.
- **Loosen the register**: delete the `## Register` section. You keep the
  structural rules (result first, one next action, capped lists) without the
  clipped prose.
- **Rename `## Worth knowing`**: it appears in rule 14 and in the pre-send check.
  Change both.
- **Do not delete `keep-coding-instructions: true`.** Custom output styles leave
  out Claude Code's built-in software-engineering instructions unless that
  frontmatter line is present. It is already set in the shipped file. Drop it while
  editing and Claude gets terse *and* worse at coding.

## Credits

- [ayghri/i-have-adhd](https://github.com/ayghri/i-have-adhd) by Ayoub Ghriss, the
  original this is derived from. Star that one too.
- Upstream in turn credits *The Adult ADHD Tool Kit* by J. Russell Ramsay and
  Anthony L. Rostain.

Related, if you want to compare ports:
[nextor2k/hyperfocus](https://github.com/nextor2k/hyperfocus),
[alexgreensh/attention-span](https://github.com/alexgreensh/attention-span).

## License

MIT. See [LICENSE](LICENSE), which carries the upstream copyright alongside this
one, as the MIT terms require.
