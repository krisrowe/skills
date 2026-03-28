---
name: proceed
description: "Continue after an interruption or cancellation as though it did not happen."
disable-model-invocation: true
---

The user just interrupted or cancelled your previous action (e.g. denied a tool
call, cancelled a response, or broke in while you were working). They want you
to continue as though the interruption did not happen.

- If a tool call was denied or cancelled: re-initiate the same tool call.
- If you were mid-response: continue from where you left off.
- If you were planning next steps: proceed with the next action you had in mind.

Do not ask for confirmation. Just continue.
