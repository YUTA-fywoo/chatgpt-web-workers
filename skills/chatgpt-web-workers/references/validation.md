# Validated capabilities

Environment: Windows Codex desktop, logged-in in-app browser, Chinese ordinary Chat UI. Checked **2026-09-11**. Treat these as observations for this environment, not promises about every future release or every file format. The user has fixed original-file uploads and generated-file downloads to the browser; re-test those routes only on an explicit user request. For other capabilities, re-test only after an interface change or a contradictory failure.

| Capability | Observation | Default behavior |
|---|---|---|
| New web Chat → direct read | Passed for two newly created conversations using the persisted ID from the browser URL. No Codex task-list lookup or refresh was used. | Read by ID directly. |
| Immediate URL after first send | Temporary `/c/WEB:...` was observed before the persisted ID. | Wait for persistence; never look up the temporary ID. |
| Direct text follow-up | Passed. The correct new reply continued the previous result, and the browser still showed 极高. | Use `send_message_to_thread`; omit Codex-only model/effort arguments. |
| Immediate read after dispatch | A previous completed reply and idle status were returned before the new reply appeared. | Match sent task marker/revision and turn; do not resend on stale state. |
| Local file link in direct prompt | Failed to transfer a file. No attachment appeared; the worker correctly reported the local path unavailable. Current send schema has no file field. | Inline suitable short text as text, or upload an actual file in the browser. |
| Browser text-file upload | Passed. The worker recovered a nonce present only inside the uploaded file and computed all three line totals and the total correctly. | Use the actual file chooser and verify content. |
| Multiple simultaneous uploads | Passed with one UTF-8 text file and one PNG image in a single message; the chooser supported multiple files. | Use multi-file selection when the current chooser supports it. |
| Multi-file content handling | Passed. The second worker identified the correct file nonce, summed the three input numbers to 131, and recognized the four image colors in order. | Verify task-relevant text and image content. |
| Direct follow-up using prior attachments | Passed. Without another upload, the second worker recovered the same nonce and values and returned the correct squared total, 17161. | Continue coherent file work by direct message in the same Chat. |
| Direct retrieval of uploaded files | Passed for both UTF-8 text fixtures and the PNG. `read_thread` returned local temporary attachment paths. All three retrieved files matched the original SHA-256 hashes. | Use returned local files before a browser download, and preserve needed files in the workspace. |
| Direct retrieval of newly generated files | Failed in the original Library text test and a fresh ZIP/TXT retest. Even with `includeOutputs: true`, the latter returned only two content-reference markers and the old input PNG/TXT attachments, with no new file bytes or usable download resources. | Go directly to browser download; do not repeat a direct-file probe. |
| Browser-generated-file download | Passed for the original Library text file via preview/download. In the fresh retest, the visible download control saved the new 45-byte TXT; SHA-256, marker, and content matched. | Follow the visible download flow and verify the local artifact. |
| General file-format coverage | Text and PNG transport were tested. Generated TXT downloads were verified. A newly generated ZIP appeared in the browser, but its bytes were not acquired during the retest; direct reception of both new ZIP and TXT failed. | Do not claim all output formats were validated; use task-specific artifact checks. |
| Exact quota reduction | Not measured. The tests establish transport behavior, not a percentage of subscription savings. | Report no invented savings figure. |

Meaningful numeric checks: first computation `37 × 19 − 48 = 655`; direct continuation `655 + 17 = 672`; file line totals `91, 116, 51` and sum `258`; independent second-file sum `131` and squared sum `17161`. No account-wide settings were changed, no Pro reasoning or Work worker was selected, and test chats were not deleted.

Current tool boundaries: `create_thread` does not expose an ordinary Chat creation target; `send_message_to_thread` exposes no attachment field; model/thinking overrides apply to Codex targets; `wait_threads` documents Codex workers. Do not invent extra fields or capabilities to work around these boundaries.

An independent web-worker review covered a short translation, stale direct reads across two workers, and a formatted-file deliverable with only a reference. The host strengthened the entrypoint's turn-matching/recovery and actual-file acceptance criteria and clarified that the Sol name does not trigger another model check. The proposed exception allowing zero workers for tiny tasks was rejected because it conflicts with the user's explicit minimum of one.
