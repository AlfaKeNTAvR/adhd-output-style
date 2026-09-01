# Install

Two things have to happen: the style file lands on disk, and the `outputStyle`
setting names it. Then `/clear`.

## Option A - let Claude do it

Paste into Claude Code:

```text
Install the ADHD output style from https://github.com/AlfaKeNTAvR/adhd-output-style, refer to the repo's AGENTS.md for instructions.
```

Claude writes the style file and sets `outputStyle` for you, after asking whether
you want it globally or in one project. Then run `/clear`: the setting is part of
the system prompt and is read at session start, so it does not apply to the
session that just edited it.

## Option B - by hand

**1. Save the style file.**

Globally, for every project:

```bash
mkdir -p ~/.claude/output-styles
curl -fsSL https://raw.githubusercontent.com/AlfaKeNTAvR/adhd-output-style/main/output-styles/adhd.md \
  -o ~/.claude/output-styles/adhd.md
```

Or for one project only, from that project's root:

```bash
mkdir -p .claude/output-styles
curl -fsSL https://raw.githubusercontent.com/AlfaKeNTAvR/adhd-output-style/main/output-styles/adhd.md \
  -o .claude/output-styles/adhd.md
```

**2. Select it.** Either way works.

*From the menu:* run `/config`, open **Output style**, select **ADHD**. This
writes the setting to the current project's `.claude/settings.local.json`, so it
covers that project, not every project.

*By editing settings:* add one top-level key. For every project, in
`~/.claude/settings.json`:

```json
{
  "outputStyle": "ADHD"
}
```

For one project, the same key in that project's `.claude/settings.local.json`.

The value is the `name:` field from the style file's frontmatter, and it is
case-sensitive: `"ADHD"`, not `"adhd"`. If a style file has no `name:` field, the
filename without `.md` is used instead.

**3. Run `/clear`,** or start a new session. The style is part of the system
prompt, read at session start, so an edit never applies to the session that made
it.

**4. Check it took.** Ask something simple. The reply should be 1-3 sentences,
open with the answer, and carry no "Let me..." and no "anything else?". Ask for
something multi-step; the reply should end with a single line starting with
`Next:`.

## Scope and precedence

`outputStyle` is honored in every settings file. Highest precedence first:

1. `.claude/settings.local.json` - this project, not committed
2. `.claude/settings.json` - this project, shared with the repo
3. `~/.claude/settings.json` - every project

So a project-local setting overrides your global one. If you set ADHD globally but
one project stays on Default, check that project's `.claude/settings.local.json`
for a leftover `outputStyle` from a `/config` run.

To try it once without persisting anything:

```bash
claude --settings '{"outputStyle": "ADHD"}'
```

## Optional - companion `CLAUDE.md` rules

The style ends with a precedence rule: it governs the **shape** of output, and on
anything else your `CLAUDE.md` wins. So two guarantees belong in `CLAUDE.md`
rather than in the style, because they must survive switching to a different
style:

```markdown
## Communication
The active output style governs length and shape. The rules below hold regardless of which style is selected, so they stay here rather than in any one style file.
- **Concerns are never cut for brevity.** Tradeoffs, risks, decisions, and clarifying questions get the same shorthand as everything else: a tradeoff is a fragment, not a paragraph. Never omit one for brevity, never expand one for importance. Never drop a unit, a threshold, or the reason for something. If I don't follow something, I will ask.
- No emojis. No en dashes or em dashes: use a hyphen, comma, colon, or sentence break. Use English.
```

The style works without these. Add them if you want the "concerns are never cut"
guarantee to hold even under another output style.

## Uninstall

Run `/config` and set **Output style** back to **Default**, or delete the
`outputStyle` key from whichever settings file holds it. `/clear` afterwards.
Delete `~/.claude/output-styles/adhd.md` if you want it gone entirely.

## Troubleshooting

- **ADHD is not in the `/config` list.** Check the path is exactly
  `~/.claude/output-styles/adhd.md`, and that the `---` frontmatter block is the
  first thing in the file with `---` on its own line above and below it.
- **Set it, nothing changed.** You are still in the session that made the edit.
  Run `/clear`.
- **Works in one project, not another.** The other project has its own
  `outputStyle` in `.claude/settings.local.json`, which outranks your global
  setting. See Scope and precedence above.
- **Replies got short but Claude got worse at coding.** The
  `keep-coding-instructions: true` frontmatter line is missing or misspelled.
  Custom styles drop Claude Code's built-in software-engineering instructions by
  default; that line keeps them.
- **Claude still opens with "Let me..." or closes with "anything else?".** Those
  are pre-send-check items. Confirm the `## Pre-send check` section survived any
  edits you made to the file.
- **Subagents ignore the style.** Expected. Output styles apply to the main
  conversation only; a subagent runs its own system prompt.
- **Concerns vanish instead of moving to `## Worth knowing`.** Add the companion
  `CLAUDE.md` rules above.
