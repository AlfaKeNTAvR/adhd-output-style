# ADHD output style for Claude Code

An **output style** that reshapes every Claude Code response: the answer in line
1, the one thing you must do or decide right under it, a hard word cap per turn
type, and every decision-changing fact inside the first 60 words.

Derived from [**ayghri/i-have-adhd**](https://github.com/ayghri/i-have-adhd) by
Ayoub Ghriss (MIT), which ships the same idea as a *skill*. This is a port to
Claude Code's output-style mechanism, plus changes for agentic (not chat) use.
See [Differences](#differences-from-i-have-adhd).

## Install

Paste into Claude Code:

```text
Install the ADHD output style from https://github.com/AlfaKeNTAvR/adhd-output-style, refer to the repo's AGENTS.md for instructions.
```

Or save [`output-styles/adhd.md`](output-styles/adhd.md) to
`~/.claude/output-styles/adhd.md` yourself.

### Then turn it on, and decide where it applies

Installing the file does not select the style. **Selecting it in
`/config` -> Output style -> ADHD writes `"outputStyle": "ADHD"` into the current
project's `.claude/settings.local.json`, so it applies to that project only.**
Start Claude Code anywhere else and you are back on Default, which reads as the
style switching itself off.

To get it in every project, write the key into `~/.claude/settings.json` yourself:

```json
{
  "outputStyle": "ADHD"
}
```

Or ask Claude to do it: "add `"outputStyle": "ADHD"` as a top-level key in my
`~/.claude/settings.json`, leaving the rest of the file untouched."

Then run `/clear`, or start a new session. The style is read at session start, so
an edit never applies to the session that made it.

Full instructions, including precedence between settings files and the optional
`CLAUDE.md` companion rules: [INSTALL.md](INSTALL.md).

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

## What a reply looks like

1. **Line 1**: the answer or result, one plain sentence, never a bullet or heading.
2. **`**Next:**`** right under it, only when you must do or decide something. One
   action or one question. Every ask and open decision lives here, never lower.
3. `---` and an optional body, only for decision-changing facts that did not fit
   above. Prose by default, at most three bullets.
4. One optional `Also: x, y, z. Ask for any.` line for everything true but not
   decision-changing: mechanism, history, alternatives.
5. `## Worth knowing` last: at most three one-line bullets tagged `Risk:`,
   `Tradeoff:`, `FYI:`. Skippable by design, so never a question or an ask.

Every reply is classified by what you asked and capped in total words: an
acknowledgement gets 15, a mid-task status 35, a yes/no question 60, a diagnosis
or completed piece of work 120, a proposal 100. "Walk me through" lifts the cap.
Only a decision-changing fact may exceed a budget, and it buys the fact, not its
explanation.

## Differences from i-have-adhd

Same core: no preamble, no recap, no closers, matter-of-fact errors, one next
action. What changed:

| | i-have-adhd | This style |
|---|---|---|
| **Mechanism** | Skill, invoked with `/i-have-adhd`, kept alive by a "Persistence" section | Output style, selected in `/config`, reinforced by the harness |
| **First line** | Lead with the *next action* | Lead with the *result*: what happened, or the answer. In an agent harness the work is usually already done, so the outcome is the news |
| **Next action** | Rule 3, end with one concrete step | `**Next:**` is line 2, directly under the answer, and it is the only place an ask, an offer, or an open decision may appear |
| **Length** | Prose rules, no numbers | Hard word caps per turn type, classified by what the reader asked, not by how much the agent found |
| **What survives the cut** | Judgment call | Defined: a fact that changes your next action, an approval, safety, something irreversible, cost, or whether the result is correct. Those land in the first 60 words with numbers and scope intact; proof, method, and identifiers stay out |
| **Non-blocking concerns** | Tangents suppressed or offered as a follow-up question | A named home: `## Worth knowing`, up to three tagged one-liners. Deferred, never dropped. Anything blocking or needing approval goes in `**Next:**` instead |
| **Work already done** | Not addressed | Never listed. The result is the news; the steps are not |
| **Banned surface** | Idioms removed in the pre-send check | Also no emojis, no em dashes or en dashes, no italic, no tables, no bold outside `**Next:**` and `## Worth knowing`, English only |
| **Background jobs** | Not addressed | When Claude Code runs the session as a background job (`claude --bg`), line 1 doubles as the job's status headline. See below |

The upstream "when to break the rules" cases (explain-fully requests, destructive
actions, real ambiguity) are kept: an explicit request for depth lifts the word
cap, and anything irreversible or needing approval counts as decision-changing.

## Background jobs

Claude Code's background jobs (`claude --bg`, listed in the agent view) run a
classifier over each reply to set the job's state and headline. The session's
system prompt asks Claude to mark a delivered task with `result:` on its own
line, a blocker with `needs input:`, and an impossible task with `failed:`.
Those markers are not documented anywhere else; they exist only in background
jobs, so in a normal interactive session you will never see one.

Rather than repeating the answer in a trailing marker line, this style puts the
marker at the start of line 1 when the system prompt asks for it: `result:` when
the ask is delivered, `needs input:` when line 1 is the one question that
unblocks Claude, `failed:` when the task is impossible as framed. The job list
then shows the real answer as the headline. Status, acknowledgement, and
proposal turns carry no marker, because a marker there would flip the job to
done while a decision is still owed.

Observed, not documented: the classifier is a language model, not a pattern
match. It has marked answers done without a marker, and marked a proposal with a
`result:` line as blocked because its `**Next:**` asked for a decision. Plain
lowercase `result:` was recognised in every logged case; bold or capitalised
variants were not tested enough to recommend.

## Requirements

Claude Code with output-style support. Nothing else: no dependencies, no hooks,
no scripts.

Tested on Claude Code 2.1.263 with Fable 5.1 and Opus 5. It should hold on any
model, since it is prompt text and nothing else, but that is the configuration it
was written against and shaped by. Reports from other models and effort levels are
welcome.

The first version of this style ran to 131 lines and made completeness absolute;
replies ballooned anyway. The current 38-line version replaced it with word caps
per turn type after five independent reviews traced the bloat to the same rules.
A metrics script that tags each reply by turn type and reports length against
the budgets lives in the author's config repo:
[`scripts/style-metrics.py`](https://github.com/AlfaKeNTAvR/.claude/blob/main/scripts/style-metrics.py).

Note: `/output-style` as a standalone command was deprecated in 2.1.73 and removed
in 2.1.91. On current versions the selector lives in `/config`.

## Limitations

- **Main conversation only.** Subagents run their own system prompt, so the style
  does not change how they respond.

## Customizing

- **Rename**: change `name:` in the frontmatter. That string is what `/config` shows.
- **Loosen or tighten the caps**: edit the numbers under `## Budgets`. They are
  total words per reply, classified by what you asked; the list of turn types is
  the only place they appear.
- **Drop a section of the shape**: delete its numbered item under `## Shape`.
  Items 3 to 5 (body, `Also:` line, `## Worth knowing`) are optional already and
  can go without touching anything else. `**Next:**` is also referenced under
  Budgets and under Renderer and register; rename it in all three sections.
- **Rename `## Worth knowing`**: it appears under Shape and twice under Renderer
  and register. Change all three.
- **Not on background jobs**: the marker sentence in Shape item 1 is inert in
  interactive sessions, since it fires only when the system prompt asks for
  `result:` lines. Leave it in or cut it; nothing else depends on it.
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
