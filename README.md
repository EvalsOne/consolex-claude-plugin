# ConsoleX

Do your [ConsoleX](https://consolex.ai) tasks with Claude and have every delivery checked against the task's saved acceptance standard before Claude finishes.

A ConsoleX task holds a method (how to do the work) and an acceptance standard (what a good result must meet). This plugin adds a task to Claude as a `/cx-…` Skill. When Claude hands over the result, ConsoleX checks it with an independent (non-Claude) judge and records the attempt in the task's run history. Claude never sees the acceptance standard, only the method, so the check stays independent. Iteration is off by default. Enable it in the task’s Checks settings to revise failed required output checks—rules and AI judgments alike—and check again, up to 1–10 revisions. Unknown results and missing external evidence need your review. Each revision and AI check incurs its usual charges. Without iteration, Claude reports failed checks for your decision.

## Where it works

The plugin does its work in hooks, so it needs an app that runs plugin hooks.

| App | What works |
| --- | --- |
| Claude Code (terminal, IDE extensions, the desktop app's Code tab) | Everything below. |
| Cowork | Connecting, adding tasks, checks and saving a session as a task. The tools of your ConsoleX tasks (the plugin's local MCP server) load only when the Cowork task runs on your own computer. |
| claude.ai chat | Not supported: chat does not run plugin hooks. |

## Requirements

- `python3` (3.9 or later). The plugin uses only the Python standard library; nothing is installed or downloaded.
- A ConsoleX account on a paid plan to connect a computer.
- Network access to `*.consolex.ai`: the plugin calls the API at `api.consolex.ai`, and tasks and Skills can link to files on `static.consolex.ai`, ConsoleX's file CDN.

### Allow ConsoleX in Cowork and claude.ai

Cowork and claude.ai run code in a sandbox that blocks domains not on its allowlist. Before you use ConsoleX there:

1. In claude.ai, open **Settings › Capabilities** (on a Team or Enterprise plan, an Owner opens **Organization settings › Capabilities**).
2. Under **Code execution and file creation** (**Create and edit files** in newer versions), add `*.consolex.ai` to **Additional allowed domains**. If the wildcard is not accepted, add `api.consolex.ai` and `static.consolex.ai` one by one.
3. Start a new task: a task that was already open may still use the old allowlist.

When the domain is blocked, `/consolex:login` says so and names this setting.

## Install

In Claude (claude.ai or the desktop app): **Customize › Plugins**, find ConsoleX and install it. It is then also available in Claude Code on the same account.

In Claude Code on its own:

```bash
claude plugin marketplace add EvalsOne/consolex-claude-plugin
claude plugin install consolex@consolex
```

Or inside Claude Code: `/plugin marketplace add EvalsOne/consolex-claude-plugin`, then `/plugin install consolex@consolex`. Restart Claude Code (or run `/reload-plugins`) afterwards. If adding the marketplace fails because you have no GitHub SSH key, set `CLAUDE_CODE_PLUGIN_PREFER_HTTPS=1` and try again.

## Use

You can type these commands, or ask Claude in plain words ("add 小语评球 to this computer") and it runs them for you. `/consolex:save` and `/consolex:logout` are the exceptions: type those yourself.

| Command | What it does |
| --- | --- |
| `/consolex:login` | Connects this computer once. Your browser opens; approve the code in ConsoleX. |
| `/consolex:status` | Shows whether this computer is connected (asked of ConsoleX), as which account and device, whether it may use your tasks' tools and files, and which added tasks cannot be checked from here. |
| `/consolex:add <words from a task name, its UUID or a task link>` | Installs the task as a `/cx-…` Skill, with the ConsoleX Skills it uses. Run it again to update. Two tasks can share a name, so every list shows each task's UUID; a UUID (or a task link) names exactly one, here and in `/consolex:start` and `/consolex:schedule`. A task whose name is in Chinese (or another non-Latin script) gets an English Skill name from Claude; ask Claude to rename it any time. With no words, it lists your ConsoleX tasks and whether each is added here (`/consolex:tasks [words]` searches the list without adding anything). |
| `/consolex:remove <task>` | Takes an added task off this computer only: its `/cx-…` Skill and the ConsoleX Skills installed only for it. The task stays in ConsoleX; `/consolex:add` brings it back. It is not for stopping a schedule: while a Claude desktop scheduled task still runs the task, it removes nothing and Claude asks whether you meant only the schedule. |
| `/consolex:start <task> [-- request]` | Does the task in this session without adding it as a Skill; the delivery is checked the same way. Words after the task name, or after ` -- `, are what you want this time; a UUID or task link ends the task name, so what follows it is the request. The command ConsoleX gives you is `<UUID> <task name>`: the name there is a label, so your session list shows which task ran, and is not a request. |
| `/cx-<task name>` | Starts the task. You can also just ask Claude for it. |
| `/consolex:check [off]` | Checks the current task's latest delivery now; `off` turns checks off for this session. |
| `/consolex:save [task name]` | Saves this session as a ConsoleX task: Claude gets the task's skill ready, then selects the actual deliverable and sends a draft with result previews. The review link opens in your external browser. Saving in ConsoleX creates only the task by default; opt in to check the selected result immediately. Saving it there adds nothing to this computer: start the task with `/consolex:start`, or run `/consolex:add` for its `/cx-…` Skill. Type it yourself. |
| `/consolex:schedule <task> <when>` | In the Claude desktop app: creates (or updates) a scheduled task without installing its `/cx-…` task Skill, for example `/consolex:schedule weekly note every Monday at 9`. Asking Claude in plain words ("run weekly note every Monday at 9") does the same through the plugin's `consolex_schedule` tool. Each run calls `consolex_start_task` and follows the latest approved method. If that method is a separate Skill, it must still be available on this computer. Run it in the folder the task should run in. A task runs from one place: when it already has a schedule in ConsoleX or on another of your computers, Claude tells you and sets up nothing until that one is off. ConsoleX shows the task's Claude schedules on its page (read-only; change them in Claude). |
| `/consolex:logout [api address \| all]` | Disconnects this computer. Type it yourself. If this computer is connected to more than one ConsoleX address (for example a self-hosted one besides ConsoleX), it lists them and disconnects nothing until you name one or `all`. If ConsoleX cannot be reached the connection is kept so you can retry; add `--local` to forget it on this computer only. You can also remove computers in ConsoleX under Settings › Coding agents. |

## Tasks with staged checks

A long task can be split into stages in ConsoleX, each with its own checks (Check settings › Staged checks). When you start such a task, the plugin tells Claude the stages and what each must hand in. After each stage Claude hands in the complete text with `consolex_checkpoint` and reads the verdict in the reply: if something did not pass it fixes the stage and hands it in again, then carries on. In ConsoleX's default **remind** mode a stage that does not pass pauses nothing and nobody waits for you; the run page shows each stage, what was not met, and that you were reminded. The one exception is a speech call whose result is unknown: it must never be repeated, so ConsoleX pauses the run for your decision. Only an action that is tied to a stage (speech generation through ConsoleX) is held until its stage has passed with exactly the text Claude sends; Claude is told "not yet", not that something broke. Bash, your browser and other integrations are not checked at a stage. When Claude finishes, the turn's result line names the required stages that did not pass; the delivery itself is checked as usual and is never held back by a stage. A stage-check execution belongs to the session that started the task (0.20.1): the plugin names this session (and an isolated run its run id) to ConsoleX, which refuses another session's execution as if it did not exist; the `PreToolUse` hook also refuses a ConsoleX tool call that carries an execution id this session did not start. An older plugin cannot hand stages in to a ConsoleX that binds them; update it.

A task that has a single required check before speech generation (the original form) still pauses for your decision in ConsoleX, as before.

## What the plugin runs, stores and sends

Everything runs locally (or in the Cowork sandbox) with your user permissions.

**Where it connects.**

- The ConsoleX API, `https://api.consolex.ai`, or the address you pass to `/consolex:login` or set in `CONSOLEX_API_BASE`. A connection or task Skill saved with `https://api.evalsone.com`, the former address of the same API, is sent to `https://api.consolex.ai`.
- Amazon S3, only in an isolated run, which no command offers (you start one with `~/.consolex/bin/consolex run '<task>'` in a terminal): the task's own files (files the task's owner uploaded to ConsoleX) are downloaded from 10-minute links that ConsoleX issues on its S3 storage (`amazonaws.com` addresses), at most 50 files of up to 50 MB each.
- `/consolex:login` opens the ConsoleX approval page on `consolex.ai` in your browser. The plugin itself does not call that site.

**Hooks.** The plugin registers hooks that run `python3 scripts/consolex.py hook`:

- `SessionStart`: writes the launcher `~/.consolex/bin/consolex` (used for isolated runs). If you have installed ConsoleX tasks, it also starts a background refresh of those task Skills, at most once an hour.
- `UserPromptSubmit`: handles the `/consolex:…` commands. When you start a `/cx-…` task it marks the session, fetches the task's latest approved method and tells ConsoleX a session started; while that session works it reports activity at most every two minutes. In any other session it exits at once without a network call.
- `PostToolUse` on `Skill`: the same as starting a task, when Claude loads a `cx-…` Skill itself.
- `PreToolUse` on `consolex_start_task`: binds the call's fresh nonce to this Claude session through a short-lived, one-use local receipt; it does not change the call or grant permission to use the tool.
- `PreToolUse` / `PostToolUse` on `AskUserQuestion`: in a task session only, tells ConsoleX that a question to you is open, then answered. In an unattended run (below) it refuses the question instead, because nobody would answer it.
- `PreToolUse` on the ConsoleX tools of a task (`consolex_run_task_tool`, and in an isolated run the task's own tools) and only on those: in a task with stage checks it adds the stage-check execution id to the call. With staged checks in remind mode that is all it does (no network call); for a task with a single check before speech generation it also asks ConsoleX whether the call may go ahead. Tools of other servers, Bash and the plugin's other tools never start it.
- `Stop`: in a task session only, submits a reply that has a "ConsoleX delivery" section for checking and shows the result. In a task with staged checks it also adds the required stages that did not pass to that result line, and never blocks because of a stage. Other sessions exit at once without a network call.
- `SessionEnd`: in a task session only, tells ConsoleX the session ended.

**MCP server.** `scripts/consolex_mcp.py` (stdio, run with `python3`) gives Claude `consolex_start_task` to start a scheduled task in this session, `consolex_task_tools` and `consolex_run_task_tool` to call the tools of your ConsoleX tasks, `consolex_checkpoint` to hand in a stage of a task with staged checks and read its result, the small tools behind `/consolex:save`, `/consolex:schedule` and the Skill rename, and `consolex_list_profiles` and `consolex_get_profile` to read your ConsoleX client and product profiles (read-only: a profile is changed only in ConsoleX), so the agent can write in a client's voice or from a product's facts outside a task. ConsoleX runs task tool calls with your connections, records them and bills them like a run in ConsoleX. Manage coding-agent tool access in that task's coding-agent run dialog. It covers only the task's selected tools and attached files, and defaults on for new ConsoleX-managed tasks; existing settings are unchanged.

**Isolated runs.** No command offers an isolated run while ConsoleX validates the product. When you run `~/.consolex/bin/consolex run '<task>'` in a terminal, it:

- creates the folder `~/ConsoleX/<task>/` and puts the task's Skills in `.claude/skills/` there and its files in `files/`;
- writes `.claude/settings.json` in that folder, which allows the ConsoleX tools (`mcp__consolex`) without a prompt and turns auto memory off, and `.claude/consolex-mcp.json`, which starts the plugin's MCP server for this task;
- starts `claude` in that folder with `--setting-sources project`, `--strict-mcp-config`, `--tools Skill,Read,Write,Edit,Glob,Grep`, `--allowedTools=mcp__consolex` and `--plugin-dir` set to this plugin, and with environment variables that turn off bundled Skills, auto memory, `CLAUDE.md` files and claude.ai MCP servers for that session.

Setting up the run changes nothing outside that folder: not your own Claude settings, Skills, plugins or memory.

**Stored on your computer.**

- `~/.consolex/credentials.json` (mode 0600): the device token from `/consolex:login`. Remove it with `/consolex:logout`.
- `~/.consolex/bin/consolex`: the isolated-run launcher.
- `~/.claude/skills/cx-…`: task Skills you add, and the ConsoleX Skills they use. Your own Skill of the same name is never replaced. Task Skills hold no secrets.
- `~/ConsoleX/<task>/`: isolated-run folders, only when you run one.
- `~/.consolex/started/`: tasks you start with `/consolex:start` without adding them, and the ConsoleX Skills they use. Claude does not load these as Skills.
- Session state in Claude Code's plugin data folder, kept for 7 days.

**Sent to ConsoleX with a delivery.** The delivery (the reply, or the text files it names), the "Needs your confirmation" items, the model id, and the tool calls of this session since the task started: commands, URLs, queries and file paths with their results. Long outputs are shortened, images are not sent, and keys and tokens are masked. Only the latest 80 tool calls are sent, so use one session per task. Set `CONSOLEX_TOOL_EVIDENCE=off` to send only the delivery.

**Sent with a stage.** In a task with staged checks, the complete text of each stage Claude hands in with `consolex_checkpoint` and the stage's name; nothing else of the session. A stage hand-in is checked by ConsoleX, which may bill the checks the task asks for.

**Fetched from ConsoleX.** Your task list (`/consolex:tasks`), a task's method and the Skills it uses (`/consolex:add`, `/consolex:start`, the hourly refresh, and when a task starts), this computer's connection (`/consolex:status`), and check results.

**Sent at `/consolex:login`.** The computer's name (on macOS its Computer Name, otherwise the host name), so you can tell computers apart on the approval page and in Settings. A background process then polls ConsoleX until you approve or the code expires (10 minutes).

**Unattended runs.** A run started by a Claude desktop scheduled task, by `consolex run --headless`, or with `CONSOLEX_UNATTENDED=1` set (for your own scheduler) never stops to ask you. Claude puts what you need to decide under "Needs your confirmation", delivers, and writes down the check result; you find it in the run record and the run email. If the files named in the delivery cannot be read, Claude fixes the paths once. When you type into such a session later, it behaves like any other.

**Sent with `/consolex:save`.** Your messages in the session, up to eight candidate results (declared file deliveries use the file contents) and the selected result, the model id, the tool records and the task's main skill (the text files of its folder; keys masked). Nothing is sent unless you run the command.

**Sent with each delivery of a task whose method is a skill.** A fingerprint of that skill; its text files too (keys masked) when they changed since ConsoleX last got them. Set `CONSOLEX_METHOD_SNAPSHOT=off` to send only the fingerprint: ConsoleX still marks runs where the skill changed, but can no longer show or read the new version. An isolated run (`/consolex:run`) of such a task copies that skill into its run folder; if another coding agent on this computer has the skill and Claude Code does not, Claude asks before copying it over.

Sessions where you have not started a ConsoleX task send nothing.

## Settings

| Variable | Effect |
| --- | --- |
| `CONSOLEX_TOOL_EVIDENCE=off` | Send only the delivery, not the session's tool calls. |
| `CONSOLEX_NO_BROWSER=1` | `/consolex:login` prints the link instead of opening the browser. |
| `CONSOLEX_HOME` | Folder for the connection (default `~/.consolex`). |
| `CONSOLEX_SKILLS_DIR` | Where `/consolex:add` installs Skills (default `~/.claude/skills`). |
| `CONSOLEX_RUNS_DIR` | Where isolated runs are created (default `~/ConsoleX`). |
| `CONSOLEX_UNATTENDED` | Set to `1` when your own scheduler starts Claude Code on a task: Claude then never stops to ask (see Unattended runs). `consolex run --headless` sets it. |
| `CONSOLEX_TOKEN` | A ConsoleX device token to use instead of the saved connection, for unattended runs such as CI. It is sent only to the ConsoleX API. |
| `CONSOLEX_API_BASE`, `CONSOLEX_APP_BASE` | A self-hosted or development ConsoleX. |

## Limits

- Deliverables are text: the reply, or UTF-8 text files changed since the task started.
- Only this session's main tool calls count as evidence; checks that need more come back as "needs review".
- Cloud routines do not load user plugins.

## Privacy and support

- Privacy policy: https://consolex.ai/legal/privacy
- Terms: https://consolex.ai/legal/terms
- Issues: https://github.com/EvalsOne/consolex-claude-plugin/issues

## License

MIT. See [LICENSE](LICENSE).

A task saved with its main skill runs in Claude Code. ConsoleX shows where it runs and a copyable `/consolex:start <task name>` command. Changed skills sync independently of deliveries, without starting a run or a billed check; each delivery still records its version. Renaming the task entry keeps its active check session.

Claude Code also limits consecutive Stop-hook continuations. To allow ten revisions in a normal terminal session, start it with `CLAUDE_CODE_STOP_HOOK_BLOCK_CAP=13` (restart an existing session). ConsoleX isolated runs set this automatically. [Claude Code hooks reference](https://code.claude.com/docs/en/hooks). The one-day save polling window is the draft review deadline, not an installation delay; an idle session shows the saved acknowledgment when you next interact.

Task deliveries now include visible model replies and tool requests in recorded order in the ConsoleX Steps timeline. Set `CONSOLEX_TOOL_EVIDENCE=off` to disable both model and tool capture.
