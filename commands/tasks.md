---
description: List your ConsoleX tasks (optionally only those whose name contains the given words) and whether each is installed here.
argument-hint: "[words from a task name]"
---

The ConsoleX plugin has already fetched the task list and added it to your context. Show it to the user as given, then say they can add one with `/consolex:add` followed by words from its name, or do one here without adding it with `/consolex:start`. Do not run any commands.

If no ConsoleX result is in your context, the plugin's hooks did not run here (claude.ai chat does not run plugin hooks; ConsoleX works in Claude Code and Cowork). Tell the user that in one line and do nothing else.
