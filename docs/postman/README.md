# Postman collection

A ready-to-run Postman collection for the Lexgo Sign public API (`/v1`): 52 requests with tests and saved examples, three environments and three fictional PDF fixtures. It covers drafts with several documents, an attachment and several recipients (signer, approver, witness), envelope metadata, presigned uploads (small/medium/large) with a bounded wait for the worker, sending, 2FA validation codes, settings, webhooks and one optional OAuth Client API call.

Every request has a description with its effect and a pointer to the matching documentation page. Folders are numbered and tagged **SAFE** (unsent drafts with fictional data) or **EFFECT** (real emails, tenant-wide changes, outbound HTTP, real Alerts).

## Files

| File | Purpose |
|------|---------|
| [LexgoSign.postman_collection.json](LexgoSign.postman_collection.json) | The collection (Postman v2.1): requests, tests, saved examples, guards |
| [LexgoSign.local.postman_environment.json](LexgoSign.local.postman_environment.json) | Environment for a local development server |
| [LexgoSign.staging.postman_environment.json](LexgoSign.staging.postman_environment.json) | Environment for staging (`api-beta.lexgo.cl`) |
| [LexgoSign.production.postman_environment.json](LexgoSign.production.postman_environment.json) | Environment for production (`api.lexgo.cl`) |
| [fixtures/small.pdf](fixtures/small.pdf) | Fictional one-page PDF, about 1 KB |
| [fixtures/medium.pdf](fixtures/medium.pdf) | Fictional one-page PDF, about 200 KB |
| [fixtures/large.pdf](fixtures/large.pdf) | Fictional one-page PDF, about 1 MB |

The PDFs contain only the text "Lexgo Sign sample - fictional"; the padding sits inside a valid PDF content stream. The application only accepts `application/pdf`. The same content as `fixtures/small.pdf` is kept as base64 in the collection variable `sample_pdf_base64` for JSON and multipart creates.

## Import

1. In Postman choose **Import** and select the collection and the environment files (or drag them in).
2. Open the environment you want, set `api_key` (and any other variable you need, see below) in the **Current value** column, and select the environment.
3. Set the working directory: **Settings > General > Working directory** must point to the folder that contains `fixtures/` (this `postman` folder). The storage uploads in folder 2 use relative paths such as `fixtures/small.pdf`, which only resolve from there.
4. Run folders in order. Folder 1 saves the ids that later folders use.

Earlier drafts of this collection used `base_url` and `client_prefix`; they are replaced by `api_url` (host only) and `app_url`. A client environment that only has `api_url` (host) and `api_key` still works, because the collection defaults `api_prefix` to `v1`.

Generated state (`envelope_id`, `presigned_url`, `presigned_url_id`, `webhook_id`, ...) lives in collection variables. See [Migrating an older environment](#migrating-an-older-environment) if you already have an environment from an earlier collection.

## Variable contract

Configuration variables are set in the environment files. Generated and state variables are defined at collection level and the collection writes them there. Scripts read every variable with `pm.variables.get` (which resolves local, data, environment, collection, then global) and write generated state with `pm.collectionVariables.set`.

| Variable | Defined in | Purpose |
|----------|------------|---------|
| `api_url` | environment | Sign API host only: `http://localhost:3000` (local), `https://api-beta.lexgo.cl` (staging), `https://api.lexgo.cl` (production). Requests are `{{api_url}}/{{api_prefix}}/...` |
| `api_prefix` | collection default `v1`; environment may override | Path prefix: `v1` on gateway hosts (staging, production), `api/v1` for a local server (set in the local environment) |
| `api_key` | environment (secret, empty) | Sign API key, sent as the `Authorization` header value |
| `app_url` | environment | Web host of the OAuth Client API: `http://localhost:3000`, `https://beta.lexgo.cl`, `https://app.lexgo.cl` |
| `oauth_token` | environment (secret, empty) | Doorkeeper access token, only for folder 7 |
| `enterprise_id` | environment (empty) | Integer enterprise id sent as `X-Enterprise-Id`, only for folder 7 |
| `document_id` | environment (empty) | Integer id of an existing contract document, only for folder 7 |
| `effects_confirmed` | environment (`no`) | Set to `yes` to let EFFECT folders run |
| `production_confirmed` | environment (`no`) | Set to `yes` to let folders 1 to 7 run on a production host |
| `api_key_other_tenant` | environment (secret, empty, optional) | A different API key, used only for the tenant denial request |
| `fixture_signer_email` | environment | Signer address, default `signer@example.com` (this name was chosen over `signer_email` to line up with existing client environments) |
| `fixture_approver_email` | environment | Approver address, default `approver@example.com`; also the second signer in the two-signer drafts |
| `fixture_witness_email` | environment | Witness address, default `witness@example.com` |
| `sample_pdf_base64` | collection | Base64 of `fixtures/small.pdf` for base64 creates |
| `webhook_url` | collection | Webhook target, default non-routable `https://example.invalid/lexgo-sign-webhook` |
| `validation_code` | collection | Code submitted by Verify validation code, default `000000` (expected to be rejected) |
| `envelope_id` | collection (generated) | Multipart draft created in folder 1 |
| `json_envelope_id` | collection (generated) | JSON draft with Unicode metadata created in folder 1 |
| `upload_envelope_id` | collection (generated) | Draft created for the upload walkthrough in folder 2 |
| `signer_id` | collection (generated) | Signer recipient id of `envelope_id` |
| `approver_id` | collection (generated) | Approver recipient id of `envelope_id` |
| `witness_id` | collection (generated) | Witness recipient id of `envelope_id` |
| `json_signer_id` | collection (generated) | Recipient id of `json_envelope_id` (changes after the update, which replaces recipients) |
| `metadata_sent` | collection (generated) | JSON text of the metadata that was sent, compared after the round trip |
| `presigned_url_id` | collection (generated) | Latest presigned upload id |
| `presigned_url` | collection (generated) | Latest presigned storage URL; cleared after the storage PUT because it is a credential |
| `webhook_id` | collection (generated) | Webhook created in folder 6 |
| `settings_backup_json` | collection (generated) | JSON text of the current settings values read in folder 5; the restore request sends it back |
| `zero_envelope_id` | collection (generated) | Draft created without files in folder 2 |
| `reminder_alert_date` | collection (generated) | Alert date (today plus 30 days) computed for folder 7 |
| `upload_poll_attempts` | collection (generated) | Attempt counter of the folder 2 completion poll; reset when the walkthrough draft is created and cleared when the poll ends |
| `storage_skipped` | collection (generated) | Set to `yes` when a presigned storage PUT was skipped (localhost); the folder 2 completion poll is then skipped too |
| `upload_poll_max` | collection | Maximum poll attempts, default `40` (capped at 100) |
| `upload_poll_delay_ms` | collection | Pause between poll attempts, default `3000` (capped at 3000) |

Resolution order: a variable set in a request's local scope or a data file wins over the environment, the environment wins over the collection, and the collection wins over globals. An environment, data file or `--env-var` that defines a generated state variable therefore **would shadow** the collection state. The collection refuses to run in that case: the collection-level pre-request script compares each generated variable with its collection value and, on a mismatch, makes the request unsendable and fails the run with an error that names the variable. It does the same when `api_url` has anything after the host.

### Email addresses and sandbox keys

The defaults are fictional `example.com` addresses. A **sandbox** key only accepts recipient emails that belong to your own team; with the defaults, the create requests return `422` with envelope status `ERROR` and "not in the sandbox allowed email list". Set `fixture_signer_email`, `fixture_approver_email` and `fixture_witness_email` to team addresses (a **live** key accepts any address). For a bounded send smoke on staging use only the approved beta own-inbox aliases of the form `contacto+<alias>@beta.lexgo.cl`, never real external addresses.

## Migrating an older environment

An environment from an earlier collection (for example one whose scripts use `pm.environment.set` for ids) can hold copies of the generated variables, and an old `api_url` that includes a path. Either fix works:

1. **Import a fresh environment** from this folder and copy only your `api_key` (and, if you use them, `oauth_token` and `api_key_other_tenant`) into it, privately. Do not paste keys into files you commit.
2. **Or edit the old environment:** set `api_url` to the host only, with no path and no trailing slash (`https://api.lexgo.cl`, `https://api-beta.lexgo.cl` or `http://localhost:3000`), and delete these variables from it, because the collection owns them:
   - `envelope_id`
   - `json_envelope_id`
   - `upload_envelope_id`
   - `zero_envelope_id`
   - `signer_id`
   - `approver_id`
   - `witness_id`
   - `json_signer_id`
   - `metadata_sent`
   - `presigned_url_id`
   - `presigned_url`
   - `webhook_id`
   - `settings_backup_json`
   - `reminder_alert_date`
   - `upload_poll_attempts`
   - `storage_skipped`

Keep `api_key`, `api_url` (host only) and, optionally, `api_prefix`, `fixture_signer_email`, `fixture_approver_email` and `fixture_witness_email`. Without `api_prefix` the collection uses `v1`, which is correct for the gateway hosts; a local server needs `api_prefix` set to `api/v1`.

If a shadowing variable or a `api_url` with a path is left behind, the first request of the run fails with an error that names it and nothing is sent.

## Gateway and URLs

Rails serves the API under `/api/v1`. The API Gateway strips `/api`, so with `api_url` as the host and `api_prefix` as the path prefix the external URLs are:

- staging: `https://api-beta.lexgo.cl/v1` (`api_prefix` = `v1`)
- production: `https://api.lexgo.cl/v1` (`api_prefix` = `v1`)
- local development server: `http://localhost:3000/api/v1` (`api_prefix` = `api/v1`, set in the local environment)

A `404` on `/api/v1/...` through a gateway host means the prefix was not stripped: use the documented form above, do not guess. The key stage (sandbox or live) belongs to the API key, not to the host. The OAuth Client API in folder 7 is different: it lives on the web host (`app_url`) under `/api/client`.

## Safety map

| Folder | Tag | Requests | What it does |
|--------|-----|----------|--------------|
| `0. SAFE - Connectivity` | SAFE | 3 | Liveness (no key), `GET /test`, `POST /test`. Only folder that runs on production without `production_confirmed` |
| `1. SAFE - Draft envelopes (unsent, fictional)` | SAFE | 20 | Auth denial (401 twice); create a draft by multipart (2 documents, 1 attachment, 3 recipients, SIGNATURE/APPROVER/WITNESS placements); create by JSON with nested Unicode metadata; get; round-trip check; update (recipients and placements; then documents and placements); six small role and file variants (1 signer/1 file with optional recipient fields and settings overrides, 1 signer/2 files with a token placement, 2 signers/2 files with numeric and custom recipient keys, 2 signers/2 files/1 attachment, approver + signer, witness + signer); scoped and full settings (read-only); tenant denial (404 with another key); three invalid-metadata cases (array, string, depth 6) |
| `2. SAFE - Uploads (draft only)` | SAFE | 15 | The supported presigned flow: a draft with a placement per file to be uploaded (`document_key` equals the presigned `custom_key`); presigned create (with `custom_key`), storage PUT and status for small, medium and large; presigned list; a bounded poll that waits until the worker has finished (4 documents, 4 linked placements, 3 uploads `COMPLETED`); a draft with 0 files followed by a presigned slot with `file_name`, `order` and `custom_key` |
| `3. EFFECT - Send and void (real emails/state)` | EFFECT | 3 | `PUT /send` emails recipients, `PUT /void`, `GET /evidence` |
| `4. EFFECT - 2FA validation codes (sends a code)` | EFFECT | 2 | `send_code` emails a code, `verify_code` |
| `5. EFFECT - Settings update (tenant-wide)` | EFFECT | 3 | Read (saves a backup), update branding, documents, emails, localization, signature and signature_flow, restore from the backup |
| `6. EFFECT - Webhooks (outbound HTTP to a URL)` | EFFECT | 5 | Create (with `auth_type`, `auth` and several `subscriptions`), list, get, update, delete a webhook pointing at `webhook_url` |
| `7. EFFECT - OAuth client reminders (optional, creates a real Alert)` | EFFECT | 1 | `POST {{app_url}}/api/client/documents/{{document_id}}/reminders` |

Guards:

- Every EFFECT folder has a folder-level pre-request script that calls `pm.execution.skipRequest()` unless `effects_confirmed` is `yes`.
- A collection-level pre-request script skips every request outside folder 0 when the host of `api_url` or `app_url` is `api.lexgo.cl` or `app.lexgo.cl`, unless `production_confirmed` is `yes`. Running the whole collection against production therefore only runs folder 0.
- Skipped requests do not run their tests; the Postman console and newman log why.

### Folder details

- **Folder 1.** The main multipart create saves `envelope_id` and the recipient ids; the small variants only assert their own response. Optional recipient fields: the API reads the phone from `phone_id` (and `phone_label`) and does not echo it back; `tax_id`, `tax_label`, `rep_id`, `rep_label` are echoed. A `token` placement has no coordinates: the token is searched as text in the PDF (the fixtures contain `[[SIGN-HERE-1]]` and `[[SIGN-HERE-2]]`). Update replaces all items of the kinds it sends (documents, recipients, placements); omitted kinds are kept and `metadata` is ignored on update. Only SIGNATURE placements take `coordinates`; APPROVER and WITNESS placements must not. The invalid-metadata requests are rejected with `422 Invalid metadata` and no `envelope` in the body. That no database row is left behind is not provable from Postman; it is proved separately by a database count in the local proof. Metadata limits: 8192 bytes serialized, 5 levels (objects and arrays each count, the root is level 1), 100 keys.
- **Tenant denial.** Envelopes belong to the API key that created them. Set `api_key_other_tenant` to any different key (ideally of another enterprise); the request is skipped while it is empty.
- **Folder 2 (supported default).** Presigned uploads allow 50 MB. Each storage `PUT` is an **outbound call to the storage host**, sends the fixture as the raw body with `Content-Type: application/pdf`, uses request auth `noauth` and carries **no** `Authorization` header (the collection has no collection-level auth, and a test asserts the header is absent). The presigned `PUT` requests are skipped when `api_url` points to localhost because the local dev server cannot complete a real storage upload. A draft created without files is answered with `422` and status `ERROR` ("no PDF files"), but it still accepts presigned slots; the slot's `custom_key` matches the placement's `document_key`. The upload walkthrough draft works the same way: it has one placement per file to upload (`document_key` `fixture-small`, `fixture-medium`, `fixture-large`) and is therefore answered with `422`/`ERROR` until the uploads complete; the worker then links each placement to the uploaded file whose `custom_key` (or id, or numeric position) equals the placement's `document_key`. Placement storage is verified on a local server; the linking after a real storage upload is verified only by the poll when the PUTs actually run.
- **Pending, completed, timeout.** Right after the PUTs the uploads are `PENDING` or `UPLOADED` and the draft is still `ERROR` (placements point at files that are not there yet). An asynchronous worker then creates the files and links the placements. The request "Wait for uploads to complete" reads the draft and its uploads and repeats itself, at most `upload_poll_max` times (40 by default) about 3 seconds apart, until the draft is `CREATED` without errors, has 4 documents and 4 placements that are all linked to a document, and 3 uploads are `COMPLETED`. It fails at once on a `FAILED` or `EXPIRED` upload, and after the last attempt it says whether uploads are still pending (a PUT or the worker did not finish) or relationships are missing. `setNextRequest` repeats only in the Collection Runner and newman; in a manual single Send, press Send again until the assertions pass. Before a manual Send in folder 3 (EFFECT), wait until the poll passes. On localhost the PUTs are skipped (`storage_skipped` is set to `yes`) and the poll is skipped too. The uploads list and the presigned list return the same records for an envelope.
- **Legacy streaming uploads are retired.** The older Lambda and API Gateway upload path (`POST /envelopes/{id}/uploads` and a `PUT` to the `upload_url` it returns) is not part of this collection. The owner decision in issue #1207 retains presigned URLs and retires that path; its returned `upload_url` also points at a legacy host that does not resolve on beta. The presigned flow in folder 2 is the only supported upload flow. The old request shapes are kept only as a retirement reference in the repository file `docs/archive/postman/legacy-lambda-streaming-uploads.md` (not runnable, not importable).
- **Folder 3.** Send emails real recipients. Void only works for `IN_PROGRESS` envelopes, so right after Send a `405` is expected and accepted by the tests; evidence needs a `SUCCESS` envelope.
- **Folder 5.** Changes configuration for the whole API key. Run the three requests in order; the read request clears any earlier backup and saves the current values of every field the update touches and the last request sends them back. Update and Restore are skipped unless a complete backup exists, and Restore builds its JSON body in a script so quotes, backslashes, newlines and double-brace text in a value stay intact. The update uses the documented default values, so it only changes tenants that customised them.
- **Folder 7.** Optional and isolated. It uses the OAuth Client API, not the Sign API: a Doorkeeper access token (scope `client`) in `oauth_token`, the integer `enterprise_id` in `X-Enterprise-Id`, and an enterprise permission on the document. It creates a real Alert and reminder; the email is sent on the fire date (alert date minus 3 days). It is skipped while `oauth_token`, `enterprise_id` or `document_id` is empty.

Usage-report export is an enterprise Settings UI action (see [Usage reports](../guides/usage-reports.md)); there is no API or Postman request for it.

## Running with newman

Install newman (`npm install -g newman`) and run from this folder. Pass secrets without writing them into files that you commit, for example with `--env-var "api_key=$LEXGO_API_KEY"`.

Safe folders against a local development server:

```bash
cd api/docs/postman
newman run LexgoSign.postman_collection.json \
  -e LexgoSign.local.postman_environment.json \
  --env-var "api_key=$LEXGO_API_KEY" \
  --working-dir . \
  --folder "0. SAFE - Connectivity" \
  --folder "1. SAFE - Draft envelopes (unsent, fictional)" \
  --folder "2. SAFE - Uploads (draft only)"
```

Staging uses the same command with `LexgoSign.staging.postman_environment.json`. Production safe check (folder 0 only):

```bash
newman run LexgoSign.postman_collection.json \
  -e LexgoSign.production.postman_environment.json \
  --env-var "api_key=$LEXGO_API_KEY" \
  --folder "0. SAFE - Connectivity"
```

To run an EFFECT folder you must opt in explicitly, for example `--env-var effects_confirmed=yes --folder "3. EFFECT - Send and void (real emails/state)"`. On production folders 1 to 7 additionally need `--env-var production_confirmed=yes`.

Working directory and file access: `--working-dir .` (the default is the current directory) makes `fixtures/small.pdf` resolve. Files inside the working directory are read without extra flags; `--insecure-file-read` is only needed for paths outside it, which this bundle never uses.

Throttling: the API rejects more than 60 requests per minute from one IP address; do not run several newman processes in parallel.

## Known limits

- The real storage `PUT` and the completion poll are not exercised against real storage and the worker by the local proof; those requests are skipped on localhost and were checked against a local stand-in server for headers, file bodies and polling behaviour.
- EFFECT folders are not run in CI or in the local proof. Their tests and examples follow the controller behavior and the public docs.
- Drafts created by the SAFE folders remain in the account; the API has no delete endpoint for envelopes.
- Base64 documents in create and update requests are limited to 5 MB decoded; use uploads for larger files.
- `POST /webhooks` with an unsupported `auth_type` currently answers `500` instead of a validation error, so the collection has no negative webhook request.
- Webhook delivery bodies and 2FA email contents are not asserted; check them in your own endpoint and inbox.
- The OAuth Client API base on the gateway (`/client`) is not verified; the collection uses the web host `app_url`.

## Maintenance

Before promoting any legacy route or request shape from an old PR, an old collection or the source back into this collection, check the owner decisions (for uploads, issue #1207) and the runtime. Retired shapes are kept only as a retirement reference, never as active requests.
