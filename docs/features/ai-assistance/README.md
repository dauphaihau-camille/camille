# AI Assistance

Camille AI Assistance is a workspace chat surface with optional explicit document attachments. The first release supports user-private AI conversation sessions, attached-document summarize behavior, structured streaming assistant responses, AI trial-pool gating, and appending completed assistant response block payloads to the active editable document.

AI Assistance is not workspace retrieval, autonomous editing, selected-text editing, or a document-only transform endpoint. The product boundary is explicit user action: AI can generate a response or draft, but document content changes only when the user applies or appends generated content.

## Goals

- Let workspace members use document-aware AI chat with explicit, removable document context.
- Preserve useful conversation history without retaining hidden raw document source snapshots.
- Enforce document access and workspace AI billing before provider calls.
- Apply AI output to documents only through explicit user actions.

## Scope

### In scope

- Workspace AI Chat hosted by the workspace-level AI shell.
- New AI Chat Draft before the first persisted AI Conversation Session.
- User-private AI Conversation Sessions, AI Chat Turns, persisted user messages, and persisted assistant responses.
- Optional AI Document Attachments from the active document route.
- Attached-document summarize as the first AI Text Transform behavior.
- Structured Streaming AI Responses.
- AI Conversation Title generation or derivation after the first successful assistant response.
- AI Conversation Grouping by recent activity.
- Subscription Plans AI Response Gate, Workspace AI Trial Pool reservation, consumption, release, and expiry.
- AI Trial Counter and Upgrade Prompt for Free Plan and Plus Plan workspaces.
- AI Append Action for completed persisted assistant messages from an active editable document route.
- AI-assisted provenance metadata for append actions.

### Out of scope

- Workspace RAG, workspace search, or implicit retrieval across workspace documents.
- Customer-facing chatbot behavior.
- External document collaborators or public viewers using Workspace AI Chat.
- Selected-text AI actions or selection-toolbar AI.
- Cursor insertion, selected-range replacement, or automatic document edits.
- Full Markdown import, rich provider-owned document JSON, embeds, mentions, comments, or complex editor state beyond the backend-normalized basic response block payload.
- Implemented translate behavior. Translate remains a disabled Pending Translation Action until target-language selection and replace-body apply behavior are designed.
- Storing raw prompt/source snapshots or unapplied generated transform draft text as long-lived hidden history.
- Plus Plan as ongoing AI access after the trial pool is exhausted.

## Users

- Workspace members who can use Workspace AI Chat.
- Workspace members with readable access to an attached document when document-aware context is used.
- Workspace members with edit permission on the active document when appending an assistant response.
- Workspace owners/admins who can act on AI Upgrade Prompts when the Workspace AI Trial Pool is exhausted.

External document collaborators and public viewers are excluded from Workspace AI Chat even if they can read a shared document.

## Documents

- [End-to-end flow](./flow.md)
- [Session lifecycle](./flows/session-lifecycle.md)
- [Document context](./flows/document-context.md)
- [Streaming response](./flows/streaming-response.md)
- [Append to document](./flows/append-to-document.md)
- [Billing and retention](./flows/billing-and-retention.md)

## Related Domain And Decision Docs

- [AI Assistance Context](../../domain/ai-assistance/CONTEXT.md)
- [Document Editing Context](../../domain/document-editing/CONTEXT.md)
- [Subscription Plans Context](../../domain/subscription-plans/CONTEXT.md)
- [Retain AI Source Snapshots Only As Operational Metadata](../../adr/0003-retain-ai-transform-metadata-only.md)
- [Use Workspace AI Trial Pool For AI Billing](../../adr/0004-use-workspace-ai-trial-pool-for-ai-billing.md)
- [Document Edit Flow](../document-edit-flow.md)
- [Document Undo And Redo Design](../document-undo-redo-design.md)
- [Demo Guide](../../operations/demo-guide.md)
