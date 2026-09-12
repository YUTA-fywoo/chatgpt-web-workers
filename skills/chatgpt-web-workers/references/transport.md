# Chat transport

Use the currently exposed browser and app-tool schemas. The interfaces below were tested on 2026-09-11; do not replay capability tests for every user task.

## Create and identify a worker

Operate the in-app browser with `mcp__cua_repl.js` and its documented CUA API. Follow its first-call/initialization instructions, keep the browser binding, and reuse a task-owned tab. Do not open another browser or reuse an unrelated personal chat. Background tabs are sufficient unless the user wants to watch.

1. Open `https://chatgpt.com/` as a new ordinary Chat. Allow the page to finish loading: an initially incomplete page can omit both mode and effort controls. Check the selected **聊天** radio rather than inferring mode from the domain.
2. If the composer shows **极高**, use it without examining the model picker. Otherwise open the effort control, choose 极高, and verify the resulting visible selection. The observed Chinese control was named **思考强度** and exposed arrow-key adjustment. Follow current visible options rather than hard-coded coordinates or a fixed number of key presses; never move into Pro.
3. Submit the first useful task, not a generic readiness exchange. For a task with input files, attach them first using [files.md](files.md), then submit the prompt and attachments together.
4. Read the current tab URL. Immediately after submission it may be `/c/WEB:<temporary-id>`; let persistence finish and read the URL again after useful work. Extract the real ID from `/c/<conversation-id>` and retain the exact URL. Do not guess IDs or copy a stale sidebar link.

`cua.listTabs({browser: <known-browser-id>, emit:false})` can read the current tab URL when the simple tab binding lacks a URL method. This is browser-tab metadata, not a Codex task-list refresh. A documented richer tab binding may also provide `url()`.

Use fresh accessibility/DOM observations to ground actions. Batch deterministic clicks/paste/submit and their resulting state check, and return only task-relevant state to the host model where possible. Use full snapshots or screenshots only to resolve a real UI uncertainty. Do not poll an unchanged screenshot repeatedly.

## Direct reads: no task-list refresh

Call `mcp__codex_app__read_thread` with the persisted ID when the first read is due; no preliminary task-list lookup is needed:

```text
threadId: <conversation-id>
turnLimit: 1
maxOutputCharsPerItem: <brief for a state probe; adequate for the complete result when receiving it>
includeOutputs: false for ordinary text; true if tool/file detail is needed
```

Web-created chats were readable by ID without `list_threads`, navigating the Codex sidebar, or refreshing the Codex conversation list. Keep this as the default. Do not impose a refresh step as preparation for every read or send.

The tool can return JSON inside `content[].text`; parse structured content or that JSON before returning it to the host model. Retain the turn/message IDs and distinguish user messages, assistant messages, attachment entries, and status. Titles and previews are not reliable completion signals.

Separate waiting from receiving the deliverable:

- **State probe:** use the latest turn and a small excerpt, then return only IDs, task/revision match, status, and whether new content exists. Cache any returned body without forwarding old answers or every partial-generation fragment. A completion-like status signals that the full result should be fetched; it is not final acceptance.
- **Result receipt:** once a new answer appears complete, retrieve its full text with an adequate supported limit, match the dispatched task/revision, and forward the complete result plus needed attachment entries. Preserve the long report when it helps host synthesis; do not replace it with a summary or request a shorter rewrite merely to save reading tokens.
- **Later reads:** omit already delivered text, echoed prompts, duplicate previews, and unchanged attachments. Return genuinely new material, or a clearly identified revised result. A stable message ID alone does not prove unchanged content: check a content fingerprint or compare cached text, and distinguish a short probe from a full result. Do not invent append-only diffs for rewritten answers.

Use `functions.exec` or another supported composition layer to cache and project tool results before they enter host context. Keep raw results outside that context for recovery. Track the last fully delivered turn/message, revision, and content fingerprint in the existing worker state; mark delivery only after the complete text has been forwarded. If a cache or context was lost, recover the needed result from the saved copy or known conversation instead of suppressing unread material. When reconsidering a known detail, read its relevant cached section rather than automatically replaying the whole conversation. This reduces repeated host input; it does not make tool calls or initial full reports free.

An immediate read after a send can still return the previous completed turn and an idle thread. In the live test, a later read returned the correct new turn without resending or refreshing the Codex list. Check the unique task ID/revision and matched user/assistant turn before accepting the answer.

If output is truncated or missing an expected ending/field, retrieve the relevant turn with an adequate supported limit. If the tool cannot return the entire message, use the supported browser or file route to obtain the missing content. Do not mark a clipped answer as fully delivered or silently summarize away required evidence. Pagination cursors retrieve older turns; do not assume they page within a single message.

## Direct text sends

Use `mcp__codex_app__send_message_to_thread` with only the known `threadId` and the human-readable `prompt` for a ChatGPT conversation. Omit `model`, `thinking`, and an invented `hostId`: model/effort overrides are supported for Codex tasks, not ordinary Chat. Preserve the verified conversation settings.

The tested send returned `{threadId: ...}` before the new turn appeared in reads. This acknowledges dispatch, not delivery/completion. Record the task marker and prior turn ID before dispatch. After useful local work, read the conversation and confirm the new task and result. Do not immediately resend on a stale read, ambiguous acknowledgement, or temporary timeout.

The same conversation remained at 极高 after a direct send. This was checked once in the browser during validation; do not repeat browser verification after every routine message. Recheck if the conversation was recreated, the user changed settings, or UI/tool evidence suggests a mode change.

The `create_thread` tool is not a substitute for creating ordinary Chat: its current targets are Codex and ChatGPT Work. `wait_threads` currently documents Codex targets, so use bounded `read_thread` checks for these Chat workers unless future tool documentation adds Chat support.

## Scheduling and recovery

- Prefer one task packet and one result per worker. Follow up for material gaps, not to request repeated progress reports.
- Submit all ready independent tasks before waiting for any result. Browser setup/submission stays sequential; the submitted Chat workers can run concurrently. Process available results without requiring every worker to finish first.
- Schedule the first state probe for when useful progress is plausible: tens of seconds for short tasks, roughly one to two minutes for substantial research. After unchanged probes, lengthen the interval; do not immediately repeat the same read. Keep individual waits interruptible and at most 60 seconds, while maintaining user updates. A short wait or an assistant progress update is not itself a reason to poll again.
- Batch due, independent state probes with the orchestration tool. Skip completed workers until a follow-up is dispatched or a specific reread is needed. Do not ask workers for separate progress reports, reread a result in the browser after direct receipt, or take full-page snapshots to check unchanged state. Use targeted current UI state only when direct transport or a required browser operation needs it.
- Save progress before context transitions. For long/multi-chat/file tasks, maintain the compact state file described in SKILL.md, including pending send markers and actual persisted IDs. Do not put full chats or secrets into it.
- If direct lookup fails, first check whether the URL still has a temporary ID or has changed, then allow one spaced synchronization retry. Only use a task-list lookup/refresh if that evidence shows it is necessary. Never make it a recurring prerequisite.
- If sending has an ambiguous result, inspect the target chat for the marker before attempting a fallback. If absent after a reasonable synchronization check, send through the browser once and verify. Do not create duplicate messages.
- A browser tab may not show a message sent outside that tab until reloaded. Reload that specific tab only when a browser operation needs current content; this is different from refreshing the Codex task list. Do not reload a tab with a staged upload, unsent draft, or ongoing generation.
- When a page changes, rediscover its controls from current UI. When a session expires, use the normal authorized login/handoff flow. Do not bypass CAPTCHAs, account limits, security interstitials, or switch to Work/Pro to recover.
- Keep completed useful work if another worker fails. Resolve only the missing part within the six-chat limit. Report an unavoidable blocker accurately rather than marking the whole task complete.

Agent-created tabs may close at turn end. For a needed continuation call the documented `markHandoff()`; for a user-facing browser deliverable call `markDeliverable()`. Store conversation URLs independently. Do not delete the corresponding chats as automatic cleanup.
