# Usage reports

Billing API clients receive Excel usage reports for their FES (simple electronic
signature, `INTERNATIONAL`) and FEA (advanced electronic signature, `QUALIFIED_CHILE`)
signatures. This page describes what the detail rows contain so you can reconcile them
with your own records.

## Workbook layout

Each report has two sheets:

| Sheet | Grain | Contents |
|-------|-------|----------|
| *Resumen mensual* | One row per day, plus pricing tiers and totals | Aggregates only: counts, tier prices, discount and totals |
| *Detalle de firmas* | One row per signed envelope in the period | Signing date, creation date, name, document, signer and signature counts, and **Metadata** |

Reports use UTC days. An envelope belongs to the period in which its last update falls
within the report window, exactly as before.

## Metadata column

The last column of *Detalle de firmas*, **Metadata**, contains the
[`metadata`](../api/envelopes.md#envelope-metadata) object you sent when you created the
envelope, written as one line of JSON text.

| Case | Cell |
|------|------|
| Envelope created with metadata | The JSON object, for example `{"customer_id":"customer-123","cost_center":"sales"}` |
| Envelope created without metadata, or before metadata existed | Empty |

- One envelope is one row, so its metadata appears once. If an envelope ever produced more
  than one detail row, each row would repeat the same object.
- The summary and pricing sheet carry no metadata. Totals are never grouped or changed by
  metadata, and metadata never affects the billed amount.
- The cell is text. Spreadsheet applications will not evaluate it as a formula or open it
  as a link, and Unicode and nested values are kept.
- Key order inside the object is Lexgo's storage order, not necessarily the order you sent.
- There is one column. Lexgo does not create a column per metadata key. Parse the cell and
  read the keys you defined.

```python
import json
from openpyxl import load_workbook

sheet = load_workbook("usage-report.xlsx")["Detalle de firmas"]
header = [cell.value for cell in sheet[1]]
metadata_col = header.index("Metadata")

for row in sheet.iter_rows(min_row=2, values_only=True):
    raw = row[metadata_col]
    metadata = json.loads(raw) if raw else {}
    print(row[2], metadata.get("customer_id"))
```

## Scheduled and requested reports

The report Lexgo generates each month for billing clients and any report generated on
request come from the same generator. Both contain the same Metadata column and the same
totals for the same data.

Metadata cannot be added to envelopes after they were created, and there is no
retroactive enrichment: rows for envelopes that never carried metadata stay empty.

## Requesting a usage report yourself

Company administrators can request one key's usage report from **Settings -> API keys**.

1. Open the row's three-dots menu and choose **Export usage report**. The action is
   available for active keys and for keys revoked within the last 30 days (a revoked key
   can export its history but can never authenticate again).
2. The side panel shows the key's label, masked value and environment, the period choice,
   the recipient and the reporting convention.
3. Choose **Month to date** or a completed previous month (earlier years are allowed; the
   only limit is the data available). Future months and rolling windows do not exist.
4. Choose **Request report**. The reply only says the report is **queued**: it is
   generated afterwards and sent by email. Delivery is not confirmed at that moment.

### Who receives it

The report is sent only to the **verified primary email of the person who requested it**.
The recipient is read on the server; there is no way to enter another address, company or
key. Requests for one key only, never for a whole company.

### Period and cutoff

- Days are **UTC days** and each envelope is counted on the UTC date it was last updated,
  exactly like the monthly report.
- **Month to date** runs from the first day of the current month through an exact cutoff
  instant captured when the request is accepted. The cutoff includes the current partial
  UTC day (and also works on the first day of the month). It appears in the
  acknowledgment, in the email and in the workbook file name (`hasta YYYY-MM-DD HH:MM
  UTC`). Records updated after the cutoff are not included.
- The period and cutoff are fixed at acceptance, so a job that runs later does not move
  them. They are *not* a frozen copy of the data: as in the monthly report, an envelope
  that is updated again later is counted by its latest update time.
- A previous month is a full calendar month with no cutoff.

### What you receive

The same FES and/or FEA workbooks as the monthly report (same eligibility, discount and
billing configuration for the key), attached to a separate customer email. Nothing is
stored publicly or reachable by link. A key with no usage report configuration (sandbox
keys, no configuration, both reports disabled, not billable in that period) is refused at
request time with the reason.

| Email | Meaning |
|-------|---------|
| Report with workbooks | Complete export |
| "No signatures" subject | The period genuinely had no signatures; the empty workbooks are attached |
| `INCOMPLETE` subject | At least one workbook failed; the email lists which. Request again to retry |
| "Could not be generated" | Nothing was produced and no workbook is attached |

Attachments are limited to 7 MB in total; larger exports fail with that reason.

### Authentication availability

If Lexgo cannot durably record that a key was used (its usage record and its safety record both fail to
save), the request is refused before it runs with HTTP `503` and
`{"error":"Authentication temporarily unavailable. Retry shortly."}`. This is not an invalid-credential
response and does not count as use: retry after a short delay.

### Retries and duplicates

Repeating the same submission (for example a network retry) returns the original
acknowledgment and never sends another email. A deliberate new request is a new export.
Delivery cannot be guaranteed exactly once: if Lexgo cannot confirm that the email
provider accepted a send, the request is marked as unconfirmed and is **not** resent
automatically. A new export for the same key and period then says the earlier send may
already have been delivered.

## Signed documents and Last used columns

The API keys list also shows:

- **Signed documents**: the all-time number of distinct documents attributed to the key
  whose signature is complete, counted once regardless of how many people signed. A fully
  signed document inside an envelope that still has other documents in progress counts.
  Envelopes, signatures, recipients, attachments, unfinished, failed or cancelled
  documents and the billing unit are not what this counts. Zero is shown as `0`.
- **Last used**: the latest time the key authenticated a Sign API request, including reads
  and polling and requests that returned an error after authenticating. Invalid or revoked
  credentials, listing keys, requesting a report and the export job never count. The value
  is a **one-minute** bucket (it is written at most once a minute) and is shown with the
  time zone, never as a second-level time. **Never used** appears only when Lexgo has
  complete lifetime tracking for the key and it shows no authenticated activity. After an
  explicitly owner-approved legacy-history rollout, a key with no recorded timestamp and
  incomplete historical coverage shows an em dash (—), which means its history is unknown.

Historical values come from the retained API call log. The log records a request when its
response is produced, so very old values may be incomplete. The Last used column stays
hidden by default until the rollout checks pass or the owner explicitly decides that
legacy historical gaps are acceptable. That decision does not create any historical
coverage and cannot waive an open actual tracking gap.
