---
description: Remove an added ConsoleX task from this computer only - its /cx-… Skill and the ConsoleX Skills installed only for it. The task stays in ConsoleX; /consolex:add brings it back. Not for stopping a schedule - to stop one, delete the scheduled task instead.
argument-hint: "<words from the task name | its /cx-… name>"
---

The ConsoleX plugin has already looked the task up among the tasks added on this computer and added the result to your context: either it removed the task, or it removed nothing and says why.

Tell the user what it says in one or two lines. If several tasks or none matched, show the list and ask which one; remove nothing yourself. If it removed nothing because Claude desktop scheduled tasks still run the task, ask the user whether they only want to stop the schedule or also want the task off this computer, and do only what they choose. If the result lists scheduled tasks after a removal, ask the user whether to delete them, and delete only the ones they agree to. Do not delete files yourself.

If no ConsoleX result is in your context, the plugin's hooks did not run here (claude.ai chat does not run plugin hooks; ConsoleX works in Claude Code and Cowork). Tell the user that in one line and do nothing else.
