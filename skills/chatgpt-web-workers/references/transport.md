# Chat transport

Use currently exposed browser and app-tool schemas. Historical capability evidence is in [validation.md](validation.md); this file defines the operating procedure.

## State and scheduling

Keep one compact task record with:

- Overall objective, final deliverable, and non-negotiable acceptance/authorization requirements; update these only for actual user changes. Track remaining coverage separately as results are accepted.
- Per owner: scope, input slice/quota, conversation ID/URL, current stage/batch and unique dispatch revision, dependencies, pending/delivered/accepted/blocked state, accepted result/evidence locations, unresolved gaps, and next action with its contribution to the overall goal. Record why fewer than four chats are used when applicable.
- Transport: prior turn ID, send/read status, attachments, and the timing and reply-identity fields defined below.

For multi-stage, multi-chat, long-running, or file work, persist this under `work/chatgpt-workers/<run-id>/state.json`; keep bulk replies in separate cached files. Save dispatch intent before sending, then update after confirmed submission, receipt, acceptance, or replanning and before context transitions. Recover from this record and needed cached evidence, not the latest reply alone. Keep stage receipt separate from acceptance and final-task completion.

Apply the entrypoint's independent continuation rule through one schedule across all pending revisions: service due deadlines and checks before sending ready work, then wait only until the earliest next event. Browser creation/submission stay sequential; workers generate concurrently. Batch due independent reads and skip received revisions until a follow-up or targeted reread is needed. A completed stage reporting limitations can be assessed for a focused next assignment; a stale, clipped, or still-generating reply cannot release that chat for another assignment.

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

The tool can return JSON inside `content[].text`; parse structured content or that JSON before returning it to the host model. Retain the turn/message IDs and distinguish user messages, assistant messages, attachment entries, and status. Titles and previews are not reliable completion signals.

Separate waiting from receiving the deliverable:

- **State probe:** use the latest turn and a small excerpt, then return only IDs, task/revision match, status, and whether new content exists. Cache any returned body without forwarding old answers or every partial-generation fragment. A completion-like status signals that the full result should be fetched; it is not final acceptance.
- **Result receipt:** retrieve the complete new assistant reply, match its task ID/revision and user/assistant turn to the dispatched assignment, then forward it with needed attachment entries once. A send acknowledgement, old reply, or idle label is not completion. Preserve useful long reports rather than replacing them with summaries to save tokens.
- **Later reads:** omit already delivered text, echoed prompts, duplicate previews, and unchanged attachments. Return genuinely new material, or a clearly identified revised result. A stable message ID alone does not prove unchanged content: check a content fingerprint or compare cached text, and distinguish a short probe from a full result. Do not invent append-only diffs for rewritten answers.

Use `functions.exec` or another supported composition layer to cache and project tool results before they enter host context. Keep raw results outside that context for recovery. Track the last fully delivered turn/message, revision, and content fingerprint in the existing worker state; mark delivery only after the complete text has been forwarded. If a cache or context was lost, recover the needed result from the saved copy or known conversation instead of suppressing unread material. When reconsidering a known detail, read its relevant cached section rather than automatically replaying the whole conversation. This reduces repeated host input; it does not make tool calls or initial full reports free.

If output is truncated or missing an expected ending/field, retrieve the relevant turn with an adequate supported limit. If the tool cannot return the entire message, use the supported browser or file route to obtain the missing content. Do not mark a clipped answer as fully delivered or silently summarize away required evidence. Pagination cursors retrieve older turns; do not assume they page within a single message.

## Webpage sends and controls

Send every message through the known Chat's visible composer in the Codex in-app browser, including text-only follow-ups, revisions, supplied code, and regeneration requests. Use supported browser controls for retries, refreshes, settings, and stopping generation. Internal conversation tools are read-only in this workflow: never call `send_message_to_thread` or substitute an API write when the webpage is unavailable.

Before submission, verify the target Chat and record the task marker and prior turn identity from available state. Use fresh page observations to fill the composer and activate its send control once; include required attachments in that same submission. Do not overwrite an unrelated draft or send a new assignment while the prior reply is still generating. Confirm that the matching user message appears on the page; a click or cleared composer alone is not proof of dispatch. Record the actual submission time for the existing timeout clock. This bounded dispatch check does not authorize early reply polling.

Reuse verified Chat/settings unless the chat is recreated, the user changes settings, or observed UI/tool evidence suggests a mode change. A stale internal read or ambiguous browser acknowledgement does not authorize an immediate resend; use recovery below. If webpage sending is unavailable, preserve the pending message and report the obstruction instead of switching transport.

The `create_thread` tool is not a substitute for creating ordinary Chat: its current targets are Codex and ChatGPT Work. `wait_threads` currently documents Codex targets, so use bounded `read_thread` checks for these Chat workers unless future tool documentation adds Chat support.

## Polling and timeout

Apply both schedules separately to each dispatched stage or correction revision, using actual wall-clock time from its submission. Each prompt has a unique revision even when its stage/batch changes; asynchronous continuation never changes another pending revision's schedule.

- Record `dispatched_at`, `timeout_at = dispatched_at + 90 minutes`, `next_read_at`, `next_browser_check_at`, the last delivered reply identity/fingerprint, and any refresh/error-retry action. Migrate older polling state by deriving these deadlines from the original dispatch time. ID persistence, partial progress, stale reads, refreshes, retries, other workers finishing, and context recovery never reset the clock. A genuinely new follow-up after a completed reply gets its own clock; duplicating or relabeling an unanswered assignment does not.
- **Direct reply scans:** first at minute **10**, then every **5 minutes**: 10, 15, 20, …, 85; minute 90 is the final deadline check. Use compact `read_thread` probes, batching only due workers. No routine reply scan before minute 10 or between scheduled scans.
- **Chat webpage checks:** first at minute **15**, then every **10 minutes**: 15, 25, 35, …, 85. Inspect the known task-owned Chat even if earlier reads were stale; this schedule is independent of direct scans. When both are due, do the direct scan first and skip the webpage check if the complete reply was received. Otherwise inspect the current page once and apply the recovery rules below.
- Stop both schedules for a completed revision. If a scheduled check reveals a new completed reply, retrieve it in full once using the matching rules above; that receipt is not an extra poll. Creation/dispatch verification and necessary file operations are not routine reply scans, but must not be used to monitor generation between checks.
- Keep schedules anchored to dispatch time, not the last check. If delayed, perform one overdue check per due channel, then advance to its next future slot; do not replay missed scans. Honor explicit rate-limit/retry delays without extending the deadline. Keep individual blocking waits interruptible and at most 60 seconds, without extra reads between chunks. User updates and context transitions preserve both next-check times and the deadline.
- Before minute 90, absence of a reply alone does not authorize cancellation, duplicate dispatch, or host takeover. Use only the recovery actions below, do useful unassigned work, or wait. User cancellation and higher-priority constraints still apply.
- **At minute 90**, check completion immediately, before any further retry or other task work. Inspect the known Chat once if the direct result is stale or ambiguous, even though this is outside the routine webpage schedule. If completion is verified, receive the reply. Otherwise—absent, incomplete, or unverifiable—**interrupt the entire current task**: preserve available results, URLs, elapsed time, observed status, and unfinished scopes, notify the user, and end the turn. Do not refresh, retry, extend waiting, dispatch replacements, perform host research, or continue other task work after this final check. A supported stop of task-owned generation is allowed; do not delete chats. Resume only on explicit user instruction; a late reply is not authorization.
- A tool timeout or idle label is not the 90-minute task timeout. A completed reply reporting inability is a returned result handled under the entrypoint's evidence-collection boundary.

User-specified timing or recovery instructions take precedence. General persistence, efficiency, or completion language does not change these schedules or bypass the timeout stop.

### Webpage recovery

At each due webpage check, match the current task/revision and inspect its actual visible state:

- **Visible error:** follow the page's stated recovery action, including any “Retry”/“Try again” control and required wait. Record the error, action, and any `retry_not_before` time. A timed retry explicitly requested by the page may occur when due between routine checks, but never at or beyond the task deadline. Use the supported control in the same Chat, not a duplicate manually pasted prompt. If no actionable instruction is given, apply the no-output condition below when applicable; otherwise preserve the error and report the limitation.
- **No result output and no visible thinking/processing/generation:** refresh that Chat page once using the supported browser action. Judge the current assignment, not older replies. A processing notice, active thinking indicator, or active generation/stop control counts as ongoing work; absence of the literal word “thinking” is insufficient.
- **Thinking/generating or output already present:** do not refresh merely because the reply is slow, partial, or a direct read is stale. Receive a complete result; otherwise keep the original schedules. An explicit page error is handled by the preceding error rule.
- After a refresh or retry, allow the page to load and make one bounded state check to confirm the action and receive a result if present. No immediate repeated refresh/retry loop. If the same error persists without a stated retry time, wait until the next scheduled webpage check. Do not reload a staged upload or unsent draft; preserve it and report the obstruction. UI recovery never changes scope, mode, ownership, or the 90-minute deadline.

## Recovery and handoff

- If direct lookup fails, use the next scheduled webpage check to confirm the persisted URL/ID and synchronize; do not add off-schedule reply probes. Only use a task-list lookup/refresh if evidence shows it is necessary, never as a recurring prerequisite.
- If a browser send has an ambiguous result, inspect that Chat for the task marker. If present, keep waiting; do not resend because internal reads lag. If confirmed absent after the page settles and no generation is active, submit the pending message through the webpage once and verify. If uncertain, preserve pending status rather than risk a duplicate; retain the original attempted-dispatch deadline until resolved.
- A restored or externally changed tab may need synchronization. For a pending reply, refresh only under [webpage recovery](#webpage-recovery). Outside generation, synchronize that tab when a required file/browser operation needs current content; this is different from refreshing the Codex task list.
- When a page changes, rediscover its controls from current UI. When a session expires, use the normal authorized login/handoff flow. Do not bypass CAPTCHAs, account limits, security interstitials, or switch to Work/Pro to recover.
- Keep completed useful work if another worker fails. For failures other than the task-wide timeout, return the missing part to Chat within the six-chat limit, or report the unresolved limitation under the entrypoint's evidence-collection boundary. Failed revision/hand-off messages and rate limits do not permit host research. The timeout rule instead stops the entire task until the user explicitly instructs continuation.

Agent-created tabs may close at turn end. For a needed continuation call the documented `markHandoff()`; for a user-facing browser deliverable call `markDeliverable()`. Preserve conversation URLs independently; do not delete chats or alter account-wide memory/privacy settings as cleanup.
