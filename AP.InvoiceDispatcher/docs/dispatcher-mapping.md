# AP Invoice Dispatcher — Runtime Mapping

## Main.xaml input contract

| Main input | Set from | Used by |
|---|---|---|
| `in_imapHost` | Orchestrator Text asset `AP_ImapHost` | `Process\\ValidateDispatcherConfiguration.xaml` → `in_imapHost` |
| `in_imapPort` | Orchestrator Integer asset `AP_ImapPort` | `Process\\ValidateDispatcherConfiguration.xaml` → `in_imapPort` |
| `in_mailboxAddress` | Orchestrator Text asset `AP_ImapMailbox` | `Process\\ValidateDispatcherConfiguration.xaml` → `in_mailboxAddress` |
| `in_queueName` | Text asset `AP_OrchestratorQueueName`; default `AP_Invoices` | `Process\\ValidateDispatcherConfiguration.xaml` → `in_queueName` |
| `in_storageBucketName` | Text asset `AP_DocumentBucketName`; default `ap-invoice-evidence` | `Process\\ValidateDispatcherConfiguration.xaml` → `in_storageBucketName` |

`Main.xaml` receives `out_isValid` as `isConfigurationValid` and `out_validationReason` as `configurationValidationReason`. If invalid, it logs an Error and ends without polling email, writing storage, creating a queue item, or moving an email.

## Secret mapping

Never place passwords in XAML or JSON. Create Orchestrator credential asset `AP_ImapCredential`; retrieve it only inside the future IMAP intake workflow. Use an approved OAuth/Integration Service connection instead if the mail platform requires modern authentication.

## AP_Invoices queue payload

| Queue field | Source |
|---|---|
| Reference | `<normalized MessageId>|<SHA256 attachment bytes>` |
| SourceMessageId | Mail provider stable message ID |
| SourceReceivedUtc | ISO-8601 UTC timestamp |
| SourceSender | Normalized sender address |
| SourceSubject | Redacted/truncated subject as policy requires |
| AttachmentName | Original filename |
| AttachmentContentType | `application/pdf` |
| AttachmentBytes | Attachment length in bytes |
| AttachmentSha256 | SHA-256 of attachment bytes |
| BucketName | `ap-invoice-evidence` |
| BucketObjectKey | `inbox/yyyy/MM/dd/<sha256>.pdf` |
| MailboxFolder | Source folder, normally `INBOX` |
| DispatcherRunId | Current job correlation GUID |
| SchemaVersion | `1.0` |

Do not put PDF bytes, mailbox credentials, tokens, bank details, or extracted invoice data in the queue.

## Invocation order once IMAP activities are configured

1. Validate dispatcher configuration.
2. Read email and save each attachment locally.
3. Invoke `Process\\ValidateInvoiceAttachment.xaml` with local file path.
4. Upload valid PDF to Storage Bucket and calculate SHA-256.
5. Invoke `Process\\BuildEvidenceStoragePath.xaml` with received UTC and hash.
6. Invoke `Process\\CreateQueueReference.xaml` with message ID and hash.
7. Invoke `Process\\ValidateQueuePayload.xaml` with the complete payload.
8. Add the queue item, then move/mark the email only after enqueue succeeds.