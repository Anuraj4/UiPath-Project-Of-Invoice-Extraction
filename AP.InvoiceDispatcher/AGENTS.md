<!-- discovery-metadata: cs=0 xaml=9 deps=1 -->
<!-- PROJECT-CONTEXT:START -->
# AP.InvoiceDispatcher Project Context

- Windows / Visual Basic UiPath process project. Main entrypoint: `Main.xaml`.
- `Main.xaml` invokes `Process\\ValidateImapIntakeConfiguration.xaml` with explicit input/output mappings and ends safely when settings are incomplete.
- Helper workflows live in `Process/`; their XAML classes follow `Process_<FileName>`.\n- `Process\\UploadInvoiceEvidence.xaml` uploads a staged PDF to the configured bucket using an explicit destination directory and file name.\n- `Process\\CreateInvoiceQueueItem.xaml` creates an AP_Invoices item with the canonical 13-field specific-content contract.
- Orchestrator Dev folder: `UiPath Project`; queue: `AP_Invoices`; bucket: `ap-invoice-evidence`.
- Configuration fields awaiting values: IMAP host, port, mailbox, source/processed folders, local staging folder, and credential asset `AP_ImapCredential`.
- Actual IMAP mailbox activities are deliberately deferred until `UiPath.Mail.Activities` is installed and its local activity docs are available.
- Canonical AP_Invoices payload contract is in `docs/dispatcher-mapping.md`; never put PDF bytes or credentials in a queue item.
<!-- PROJECT-CONTEXT:END -->