---
description: Get the terminal command for an isolated run of a ConsoleX task - a new Claude Code session with only the task's Skills, files and ConsoleX tools. To do a task right here, use /consolex:start.
argument-hint: "<words from the task name>"
---

The ConsoleX plugin has already added the terminal command for an isolated run of this task to your context.

Tell the user what it says in a few lines, including the command exactly as given. Do not run the command yourself and do not run any other commands.

If no ConsoleX result is in your context, the plugin's hooks did not run here (claude.ai chat does not run plugin hooks; ConsoleX works in Claude Code and Cowork). Tell the user that in one line and do nothing else.
