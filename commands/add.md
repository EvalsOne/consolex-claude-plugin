---
description: Add a ConsoleX task to Claude Code as a /cx-… Skill. Give words from the task name or paste its ConsoleX link; with nothing, it lists your tasks.
argument-hint: "[words from a task name | ConsoleX task link]"
---

The ConsoleX plugin has already looked the task up (and installed its Skill when exactly one task matched) and added the result to your context.

Tell the user what it says in at most two lines. If several tasks matched, show them and ask which one. Do not run any commands and do not create files yourself. The one exception: when the result asks you to give the Skill an English name, call the ConsoleX tool `consolex_rename_skill` as it says, then tell the user the new name.

If no ConsoleX result is in your context, the plugin's hooks did not run here (claude.ai chat does not run plugin hooks; ConsoleX works in Claude Code and Cowork). Tell the user that in one line and do nothing else.
