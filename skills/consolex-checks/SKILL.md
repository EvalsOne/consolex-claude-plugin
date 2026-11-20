---
name: consolex-checks
description: How ConsoleX tasks work in Claude Code - connecting, adding tasks, saving a session as a task, and how deliveries are checked. Use when a ConsoleX task Skill (a cx-… Skill) is active, when ConsoleX reports a check result, or when the user asks how to use ConsoleX in Claude Code.
---

# ConsoleX in Claude Code

## Setting up (the user does this, once per computer)

- `/consolex:login` connects this computer: the browser opens, the user approves the code in ConsoleX, and the plugin saves the connection by itself. A development or self-hosted ConsoleX takes its API address: `/consolex:login https://…`.
- `/consolex:add` with no words lists the user's ConsoleX tasks (`/consolex:tasks [words]` searches them); `/consolex:add <words from a task name>` (or a ConsoleX task link) adds one as a `/cx-…` Skill, together with the ConsoleX Skills the task uses; the task Skill names them under "Skills for this task", so load those when a step needs them. `/consolex:remove <task>` takes an added task off this computer (the task stays in ConsoleX). `/consolex:logout` disconnects the computer.
- When the user asks in plain words for one of these ("add weekly report", "remove the weekly note task"), run the command for them with the Skill tool: the plugin answers it the same as a typed one. `/consolex:logout` and `/consolex:save` are the exceptions: only the user types them.
- `/consolex:save [task name]` sends the current session to ConsoleX as a task: the method stays in one skill on this computer (the task's main skill, which the plugin's note has you choose or create and bring up to date, asking the user once before you add requirements to an existing skill), and ConsoleX keeps the delivery goal, run rules and checks. You then call `consolex_save` with the note's save id (in a session without that tool, the command the note gives); the user reviews and saves the draft in ConsoleX. Saving alone does not create or check a first run: the user must explicitly choose to check the selected result preview. Saving it there adds nothing to this computer: the user starts the task with `/consolex:start`, or runs `/consolex:add` for its /cx-… Skill. When a task's method is a skill, load that skill and follow it; if it is not on this computer, stop and tell the user (the plugin records that the run did not happen). Only the user starts it; never suggest it in place of finishing the work, and never send anything to ConsoleX any other way.
- `/consolex:schedule <task> <when>` (Claude desktop app) creates or updates a scheduled task without installing its task Skill. When the user asks for one in plain words, or no ConsoleX result came with the command, call the `consolex_schedule` tool and pass on the prompts it gives. Its prompt names the task, folder and start tool: rules about the work (what to publish, whether to wait, what to do after a failed check) belong in the task in ConsoleX, never in that prompt. A task runs from one place: when the plugin says it already has a schedule in ConsoleX or on another computer, tell the user and create nothing until that one is off (pausing or deleting the ones on this computer stays fine). Pause, change or delete scheduled tasks only through the desktop app's scheduled-tasks tools, so ConsoleX hears about it.
- These commands are handled by the plugin. Never edit `~/.consolex/credentials.json`, never ask the user for a token, and never create task Skills by hand.

## Working on a task

A ConsoleX task Skill (`cx-…`) carries one task's method and an output contract. When it loads, the plugin marks the session and adds the latest approved method to your context; follow that method.

Reading a task's `SKILL.md` alone does not activate a scheduled task. Before doing a scheduled task always call `consolex_start_task` with its UUID, `unattended=true` and the scheduler's task ID as `schedule_id` when present. In Claude, generate a fresh 32-character lowercase hexadecimal `activation_nonce` for every call. In Codex, the trusted hook supplies `_activation_context`; omit it yourself. Continue only after `status="started"` identifies the requested task and returns its latest method. If the tool is missing or activation fails, report that the task was not executed or checked; do not fall back to the local Skill or synthesize, upload, publish or send. Report an actual check result only after the plugin returns it.

You do not see the task's acceptance standard: it stays in ConsoleX so the check is independent of you. Do the work the method describes as well as you can; do not guess at or write to hidden checks. After a check you see only the items that did not pass, with their reasons.

When you hand over the result, end your reply with the two sections the output contract names, the delivery section last:

```
## Needs your confirmation
- Anything the owner must decide or verify; "None" if nothing
## ConsoleX delivery
- Deliverable: this reply        (or one "- File: path/from/the/project/folder" line per text file you wrote)
```

Write the confirmation section once in the whole reply. Many task methods already ask for a "Needs your confirmation" section: then that is the one, so do not add a second copy.

Write each decision as a choice with its options ("Register cairninbox.com alone, or with .ai and .io too?"), so it can be put to the owner as a question.

Leave both sections out while you are asking a question or reporting progress: only a reply with the delivery section is checked. The judge also sees the tool calls and results of this session, so check facts with real tools (lookups, fetches, searches) rather than stating them; printing a result yourself (for example with echo) is not evidence. For file deliverables, list only text files you wrote or changed for this task, with paths relative to the project folder (or absolute paths), even after changing folder. Plain paths, inline-code paths and Markdown links to local files are accepted. Supported UTF-8 formats include .md, .txt, .json, .jsonl, .csv and .html; each file must fit the 200 KB limit. An unreadable declared file prevents submission, so do not silently omit required evidence.

## After the check

ConsoleX checks the delivery against the task's saved standard with an independent (non-Claude) judge. When it passes and nothing needs confirmation, the plugin only shows the result.

When it passes (or could not be checked) and the delivery lists items under "Needs your confirmation", the turn continues once with a ConsoleX message that starts with the result. Then, without changing the deliverable or delivering again, give the owner the result in one line with the run link and ask about the items: with AskUserQuestion when you have it (one question per item that needs a decision, at most 4, each with 2-4 concrete options from the delivery; mention reminders in your message), otherwise as questions in text, and stop. Act on the owner's answers; if they change the deliverable, deliver again with the delivery section.

Iteration is off by default for new tasks; existing tasks keep their saved setting. The owner can enable 1–10 revisions. Each revision and AI check has its usual charge. The plugin's current ConsoleX message tells you whether another revision is authorized; do not infer authorization from a failed verdict alone.

When that message says the owner enabled iteration and asks you to revise, fix the listed output items without asking again, preserve the parts that already meet the requirements, and deliver the full result again with the delivery section. Required output failures from rules and AI checks share this loop and its saved limit. Follow each new check result; stop when the plugin reports a final result or tells you to stop. Host limits may end the loop before the saved limit. If an older server explicitly sends a rules-only revision instruction, follow that narrower scope.

Unknown results, missing external evidence and items needing the owner's decision are not permission to keep rewriting. Do not change the acceptance criteria or perform unauthorized external actions to pass. Put unresolved decisions under "Needs your confirmation".

Evidence recovery is a separate, default-off task setting. The plugin may add relevant, already captured excerpts and recheck once while keeping the original evidence and standard. Do not run new tools or rewrite the delivery just to recover evidence. If it reports an uncertain recheck, `/consolex:check` resumes that same attempt; do not start another task or submit a new delivery to retrieve its result. Unchanged evidence reuses the original check without a new check charge.

When the plugin reports failed items or asks for the owner's review without authorizing another revision (including when iteration is off or has stopped):

1. Do not change the deliverable and do not deliver again yet.
2. Tell the owner the result in one short message: each item with its reason, and the run link.
3. Ask whether to revise for these items: with AskUserQuestion when you have it (options "Revise these items" / "Keep as is"; items to confirm go into the same call as further questions), otherwise in one line, and stop.
4. After the owner says to revise, fix those items in the deliverable itself and deliver again with the delivery section. ConsoleX checks it again; follow its next instruction.

In an unattended run (a scheduled task or `consolex run --headless`), follow authorized revision instructions as above, but never use AskUserQuestion or wait for an answer. Leave open decisions under "Needs your confirmation". When the plugin reports the final result without authorizing another revision, summarize the result and run link and stop.

Never invent facts, sources or approvals to pass a check, and do not add content only because a check named it; if a failed item asks for something the method never mentioned, say so to the owner. Do not submit another attempt after a report-only result until the owner authorizes it.

- `/consolex:check` checks the latest reply now; `/consolex:check off` stops checking this session.
- Every checked attempt appears in the task's run history in ConsoleX, labelled "External · Claude Code".
