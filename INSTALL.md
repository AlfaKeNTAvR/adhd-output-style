# Install

Two ways. Both end at the same place: one file on disk, one selection in `/config`.

## Option A - let Claude do it

Paste into Claude Code:

```text
Install the ADHD output style from https://github.com/AlfaKeNTAvR/adhd-output-style, refer to the repo's AGENTS.md for instructions.
```

Claude writes the file, then tells you to do the `/config` step. It cannot do that
step for you.

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

**2. Select it.**

Run `/config`, open **Output style**, select **ADHD**. Then `/clear` or start a
new session, because the change takes effect on the next session.

The selection persists: Claude Code writes an `outputStyle` key to your settings.
You do not re-select it every session.

**3. Check it took.**

Ask something simple. The reply should be 1-3 sentences, open with the answer, and
carry no "Let me..." and no "anything else?". Ask for something multi-step; the
reply should end with a single line starting with `Next:`.

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

Run `/config`, set **Output style** back to **Default**. Delete
`~/.claude/output-styles/adhd.md` if you want it gone entirely.

## Troubleshooting

- **ADHD is not in the `/config` list.** Check the path is exactly
  `~/.claude/output-styles/adhd.md`, and that the `---` frontmatter block is the
  first thing in the file with `---` on its own line above and below it.
- **Replies got short but Claude got worse at coding.** The
  `keep-coding-instructions: true` frontmatter line is missing or misspelled.
  Custom styles drop Claude Code's built-in software-engineering instructions by
  default; that line keeps them.
- **Nothing changed after selecting it.** The change applies on the next session.
  Run `/clear`.
- **Claude still opens with "Let me..." or closes with "anything else?".** Those
  are pre-send-check items. Confirm the `## Pre-send check` section survived any
  edits you made to the file.
- **Subagents ignore the style.** Expected. Output styles apply to the main
  conversation only; a subagent runs its own system prompt.
- **Concerns vanish instead of moving to `## Worth knowing`.** Add the companion
  `CLAUDE.md` rules above.
