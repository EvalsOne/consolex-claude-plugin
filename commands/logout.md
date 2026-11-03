---
description: Disconnect this computer from ConsoleX and remove it from the account's connected devices.
disable-model-invocation: true
---

The ConsoleX plugin has already disconnected this computer (or found it was not connected) and added the result to your context. Tell the user in one line. Do not run any commands.

If no ConsoleX result is in your context, the plugin's hooks did not run here (claude.ai chat does not run plugin hooks; ConsoleX works in Claude Code and Cowork). Tell the user that in one line and do nothing else.
