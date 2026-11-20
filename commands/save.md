---
description: Save this Claude Code session as a ConsoleX task - the method stays in one skill here, ConsoleX keeps the delivery goal, run rules and checks. You review the draft in ConsoleX and save it there.
argument-hint: "[task name]"
disable-model-invocation: true
---

The ConsoleX plugin has prepared this save and added its steps to your context (a note that starts with "ConsoleX: the user ran /consolex:save"). Nothing has been sent yet.

Follow those steps exactly: choose or create the task's main skill, ask the user once before writing new requirements into an existing skill, select the actual deliverable from the captured candidates (not a task list or status reply), then call the `consolex_save` tool with the save id and result_id the note gives (if this session has no such tool, run the command the note gives instead), and tell the user in two or three lines what it says, with its review link. Explain that the review link opens in the external browser (or should be copied there), and saving only checks the selected result if the owner opts in. Do not send anything to ConsoleX any other way.

If the note says ConsoleX is not connected or there is nothing to save, tell the user that in one line and do nothing else. If no ConsoleX result is in your context, the plugin's hooks did not run here (claude.ai chat does not run plugin hooks; ConsoleX works in Claude Code and Cowork). Tell the user that in one line and do nothing else.

If the owner requested open final acceptance, pass `acceptance_mode: "open_final"` if the tool supports it. The owner can also choose it on the ConsoleX review page. A leading `--open` or `--checklist` selects the mode, not the task name.
