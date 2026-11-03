---
description: Run an added ConsoleX task on a schedule from the Claude desktop app. The scheduled prompt only says where to run and which task; what to do, what to publish and what happens after a failed check all come from the task in ConsoleX.
argument-hint: "<task: words from its name or its /cx-… name> <when, e.g. every day at 7:30>"
---

The ConsoleX plugin has already looked the task up among the tasks added on this computer and put what to do next, with the exact prompts, in your context.

Follow it: work out the schedule from the user's words, then create the scheduled task with the desktop app's scheduled-tasks tools (or update the one that already runs this task), passing the prompt exactly as given. If the plugin listed several tasks or none, show the list, ask which task, and create nothing. If it says the task already runs from somewhere else (a schedule in ConsoleX or on another computer), tell the user and create nothing.

If no ConsoleX result is in your context, the plugin's hooks did not run here: they run only when the user types the command, not when you start it yourself, and claude.ai chat runs none. Then call the ConsoleX plugin's `consolex_schedule` tool with the task part of the arguments (not the time) and follow its result the same way. If you have no such tool either, tell the user that in one line and do nothing else.
