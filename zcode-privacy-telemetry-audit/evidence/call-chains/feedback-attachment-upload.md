# Call Chain: Feedback Attachment Upload → OSS (only object-storage upload in tree)

Audit: ZCode privacy / telemetry
Commit: `872ad960de7ec172591f7e1952f7849229f94521`

Entry: User explicitly opens feedback and attaches a file (feedback ticket flow).
↓ *EVID-ATTACHMENT-002__feedbackHttpClient.ts* (packages/services/src/feedback/)
↓ `uploadFile(ticketId, kind, filePath, filename, contentType, options?)` (line 105-150)
↓
`request("/feedback/attachment/upload-credential", { method: "POST", body: { ticket_id, message_id?, file_name, size } })`
↓ server returns short-lived OSS credential: `{ oss: { host, path, policy, x_oss_signature, x_oss_signature_version, x_oss_credential, x_oss_security_token, x_oss_date }, max_size, callback: { url, body, content_type } }`
↓
`uploadOssForm(credential, filePath, filename, contentType)` (line 932-1052)
↓ POST to `credential.oss.host` with multipart form-data
↓ fields built by `buildOssFormFields` (line 1054-1075): key, policy, x-oss-signature, x-oss-signature-version, x-oss-credential, x-oss-security-token, x-oss-date, `callback` (base64 JSON: callbackUrl/callbackBody/callbackBodyType), success_action_status=200
↓ streaming file up to `max_size` (500MB configurable? actual cap enforced client-side at `getFeedbackMaxAttachmentBytes`)
↓
On success returns `FeedbackAttachment` local object.

## Trigger analysis — not a pre-send upload

- `uploadFile` is reached only from the **feedback form flow** (`FeedbackHttpClient`), i.e. a user-initiated support/report action.
- It is **not** invoked by the normal chat send path. Separately, the composer run-path attaches files via `attachmentUpload.ts` (EVID-ATTACHMENT-001) which for **local paths performs ZERO upload** (ref carries absolute path), and only inlines `dataBase64`/`textContent` (pasted images / text) through a put transaction.
- Cancellation: uploads honor `options.signal` → `FeedbackUploadCanceledError`; `req.destroy`.

## Reachability / safety

OSS host and all signatures come from the server-issued credential; no static OSS URL is embedded. Sensitive signature/security-token values must be treated as confidential — they are not persisted by evidence package.