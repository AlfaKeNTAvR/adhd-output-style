# Instructions for the agent installing this

You are installing the ADHD **output style** for Claude Code. This is not a skill
and not a plugin. Do not create a skill, do not run `claude plugin install`.

## Steps

1. Copy `output-styles/adhd.md` from this repo to `~/.claude/output-styles/adhd.md`.
   Create the `output-styles` directory if it does not exist. Copy the file
   verbatim, including the YAML frontmatter, which must be the first thing in the
   file. If you are working from a clone, `cp` it. If not, fetch it from
   `https://raw.githubusercontent.com/AlfaKeNTAvR/adhd-output-style/main/output-styles/adhd.md`.

   For a single project instead of globally, use `.claude/output-styles/adhd.md`
   in that project.

2. Do **not** hand-edit the `outputStyle` key in settings unless the user asks.
   Tell the user to run `/config`, open **Output style**, and select **ADHD**.
   The selection is written to settings for them and takes effect after `/clear`
   or in a new session.

   `/output-style` as a standalone command was removed in Claude Code 2.1.91.
   Do not tell the user to run it.

3. Ask the user whether they also want the optional `CLAUDE.md` companion rules
   in `INSTALL.md`. Do not merge anything into their `CLAUDE.md` without asking.

4. Report the two things they need: the file path you wrote, and the `/config`
   step they must do themselves.

## Do not

- Do not modify `~/.claude/CLAUDE.md` unless the user agrees in step 3.
- Do not change any other output style or overwrite an existing `adhd.md` without
  saying so first.
- Do not add hooks, scripts, or dependencies. This style has none.
