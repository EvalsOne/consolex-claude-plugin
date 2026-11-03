---
description: Do a ConsoleX task here, in this session, without adding it as a Skill - the delivery is checked like an added task's. Give words from its name or its ConsoleX link, then optionally what you want this time after " -- ".
argument-hint: "<words from the task name | ConsoleX task link> [-- what you want this time]"
---

The ConsoleX plugin has already looked the task up and, when exactly one task matched, loaded it into your context: its method, its Skills and its ConsoleX tools.

If the task was loaded, say in one line which task you are starting, then do it now as your context says. If several tasks matched, show them and ask which one; the user then runs `/consolex:start` with more words of its name. Otherwise tell the user what the result says in one or two lines and do nothing else.

If no ConsoleX result is in your context, the plugin's hooks did not run here (claude.ai chat does not run plugin hooks; ConsoleX works in Claude Code and Cowork). Tell the user that in one line and do nothing else.
