# Instructions for the agent installing this

You are installing the ADHD **output style** for Claude Code. This is not a skill
and not a plugin. Do not create a skill, do not run `claude plugin install`.

Two things have to happen: the style file lands on disk, and the `outputStyle`
setting names it. You can do both.

## Step 1 - write the style file

Copy `output-styles/adhd.md` from this repo to `~/.claude/output-styles/adhd.md`.
Create the `output-styles` directory if it does not exist. Copy it verbatim,
including the YAML frontmatter, which must be the first thing in the file. From a
clone, `cp` it. Otherwise fetch:

```
https://raw.githubusercontent.com/AlfaKeNTAvR/adhd-output-style/main/output-styles/adhd.md
```

For a single project instead of globally, write `.claude/output-styles/adhd.md` in
that project.

## Step 2 - ask the user which scope, then set `outputStyle`

Ask before writing to a settings file. Two choices:

- **Everywhere**: add `"outputStyle": "ADHD"` to `~/.claude/settings.json`.
- **This project only**: add it to `.claude/settings.local.json` in the project.

The key is flat, top level, and the value is `"ADHD"` exactly. It is the `name:`
field from the style's frontmatter, and it is case-sensitive. Merge the key into
the existing JSON; do not overwrite the file.

Note the precedence if you set both: project-local wins over user-level. A project
that already pins a different style keeps winning until you change that file too.

If the user would rather pick from a menu, tell them to run `/config` and choose
**Output style** -> **ADHD**. That writes the same key, to the project-local file.

## Step 3 - tell the user to `/clear`

The setting is part of the system prompt, so it is read at session start. Your
edit does **not** apply to the running session. The user has to run `/clear` or
start a new session. Say this explicitly; otherwise they will think the install
failed.

## Step 4 - report

Give them three things: the style file path you wrote, the settings file you
changed and the scope it covers, and the `/clear` step they must do themselves.

## Do not

- Do not write any settings file without asking which scope the user wants.
- Do not modify `~/.claude/CLAUDE.md`. The optional companion rules in
  `INSTALL.md` are opt-in; mention them, do not merge them.
- Do not overwrite an existing `adhd.md` or change another output style without
  saying so first.
- Do not tell the user to run `/output-style`. It was removed in Claude Code 2.1.91.
- Do not add hooks, scripts, or dependencies. This style has none.
