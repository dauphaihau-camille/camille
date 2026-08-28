# Camille AI Assistance

Camille AI assistance covers AI-generated chat responses, transform drafts, model-backed document assistance, provider boundaries, and AI feature surfaces across the product. It does not own document editing persistence or subscription billing counters.

## Language

**AI Document Scope**:
The current Document Title as context and current Document Body as transformable content that an AI writing action is allowed to read.
_Avoid_: Workspace knowledge, retrieved context, page content

**Workspace AI Chat**:
A multi-turn AI conversation in a workspace that may use explicitly attached documents but does not retrieve across the workspace.
_Avoid_: Workspace Q&A, RAG, autonomous agent

**AI Conversation Session**:
A user-private conversation thread within a workspace for Workspace AI Chat.
_Avoid_: Shared workspace session, document-only session, draft history

**AI Chat Turn**:
One user message and its generated assistant response inside an AI Conversation Session.
_Avoid_: Prompt attempt, token event, billing invoice line

**AI Chat Message Content**:
The persisted user message and assistant response text in an AI Conversation Session.
_Avoid_: Source snapshot, hidden prompt log, provider trace

**Streaming AI Response**:
An assistant response delivered incrementally to the AI Chat Panel before the full AI Chat Turn is complete.
_Avoid_: Fake streaming, background job

**AI Conversation Title**:
A short generated label for an AI Conversation Session.
_Avoid_: Document title, manual-only title

**AI Conversation Group**:
A recency group used to organize AI Conversation Sessions in the session selector.
_Avoid_: Document group, shared team group

**New AI Chat Draft**:
The local, unpersisted starting state shown before the first AI Chat Turn creates an AI Conversation Session.
_Avoid_: Deleted conversation, persisted empty session, hidden draft history

**New AI Conversation Action**:
The header action that returns the panel to the New AI Chat Draft without deleting existing sessions or creating a persisted empty session.
_Avoid_: Clear chat, delete session, pre-create session

**AI Document Attachment**:
A document badge attached to an AI Chat Turn as explicit context.
_Avoid_: Retrieved document, implicit workspace context, RAG citation

**AI Text Transform**:
A user-requested AI writing action that returns a transformed draft from the current Document Body.
_Avoid_: RAG, workspace Q&A, autonomous agent

**AI Transform Draft**:
A preview result from an AI Text Transform represented as basic document blocks that has not changed the collaborative document.
_Avoid_: Automatic edit, generated document, full rich document

**AI Chat API**:
The backend-owned workspace endpoint that creates AI Chat Turns and AI Transform Drafts from AI Chat Panel Actions.
_Avoid_: Browser LLM call, provider proxy, document-only transform endpoint

**AI Append Action**:
An assistant-message action labeled "Append to this document" that appends the assistant response content to the end of the current active editable document.
_Avoid_: Cursor insert, selection replace, automatic edit

**AI Append Block Conversion**:
The client-side conversion from assistant response text into supported basic document blocks before an AI Append Action writes to the editor.
_Avoid_: Full markdown import, provider-owned document JSON, rich document conversion

**AI Transform Source Snapshot**:
The server-side materialized document title and body projection used by the AI Chat API.
_Avoid_: Client-sent editor text, live Y.Doc dependency

**AI Source Query**:
The document application read model that resolves access and returns the AI Transform Source Snapshot.
_Avoid_: Document detail read, direct entity read, visit-recording read

**AI Source Snapshot Token**:
An opaque identifier for the AI Transform Source Snapshot that an AI Transform Draft was generated from.
_Avoid_: Stored source text, client text comparison

**AI Transform Metadata**:
Operational facts about an AI Text Transform, AI Chat Turn, or AI Append Action that do not include source document snapshots.
_Avoid_: Source snapshot history, provider trace

**AI Size Limit Notice**:
A notice shown when the requested AI Text Transform exceeds the direct-context size limit.
_Avoid_: Partial summary, hidden chunking, retrieval fallback

**AI Empty Content Notice**:
A notice shown when the requested AI Text Transform has no meaningful Document Body text to transform.
_Avoid_: Title-only transform, empty LLM call

**AI Draft Failure Notice**:
A retryable notice shown when an AI Transform Draft cannot be produced or validated.
_Avoid_: Malformed draft, best-effort repair

**AI Rate Limit Notice**:
A notice shown when operational AI request limits deny an AI Text Transform.
_Avoid_: Entitlement denial, upgrade prompt

**AI Chat Panel**:
The product surface that displays Workspace AI Chat sessions, document attachments, AI responses, transform progress, draft output, append/copy actions, and draft actions.
_Avoid_: Workspace Q&A panel, RAG panel

**AI Chat Panel Action**:
A panel action that starts an AI Chat Turn or AI Text Transform in the current AI Conversation Session.
_Avoid_: Selection toolbar, agent tool

**AI Document Context Registration**:
The route-local handoff that offers the current document as an AI Document Attachment to the workspace AI shell.
_Avoid_: Shell route scraping, duplicate document fetch

**Summary Shape**:
The default form of a summarize AI Transform Draft.
_Avoid_: Freeform response, exhaustive recap

**Pending Translation Action**:
A visible but disabled translate AI Chat Panel Action reserved for a later AI Text Transform.
_Avoid_: Implemented translation, hidden roadmap item, clickable placeholder

**Translation Target Language**:
The user-selected language for a future translate AI Text Transform.
_Avoid_: Inferred language, automatic locale

**Stale AI Transform Draft**:
An AI Transform Draft whose AI Source Snapshot Token no longer matches the current transform source before the draft is applied.
_Avoid_: Auto-merge, best-effort apply

## Relationships

- **Workspace AI Chat** belongs to a workspace and can run with zero or more **AI Document Attachments**.
- An **AI Conversation Session** is persisted at workspace scope and visible only to the user who created it.
- An **AI Conversation Session** contains one or more **AI Chat Turns**.
- An **AI Conversation Session** persists **AI Chat Message Content**.
- **AI Chat Message Content** is product state; raw **AI Transform Source Snapshot** text is not retained as chat history.
- An **AI Conversation Session** starts as untitled and receives an **AI Conversation Title** after the first successful assistant response.
- An **AI Conversation Title** is generated or derived from the first **AI Chat Turn**, not from an attached document title.
- The header **New AI Conversation Action** returns the panel to the **New AI Chat Draft**; it does not create an **AI Conversation Session** until the first AI Chat Turn is submitted.
- **AI Conversation Sessions** are grouped by last activity into **AI Conversation Groups** such as Past week and Older.
- An **AI Chat Turn** can use attached documents as explicit context; it does not retrieve other workspace documents.
- **AI Document Attachments** are stored per **AI Chat Turn**.
- Removing an **AI Document Attachment** affects future turns only; earlier turns keep their original attachment record.
- A chat turn with no **AI Document Attachments** is a general workspace assistant response with no document context.
- A successful **AI Chat Turn** produces one assistant response and consumes Subscription Plans **AI Response Gate** access.
- The first implemented AI assistance behavior is summarize inside **Workspace AI Chat** when a document is attached.
- A summarize **AI Text Transform** produces a short bullet **Summary Shape** by default.
- Translate remains a **Pending Translation Action** until the language picker, translate intent, and replace-body apply behavior are implemented.
- A **Pending Translation Action** is disabled with coming-soon copy; it does not call the **AI Chat API**.
- A future translate **AI Text Transform** will require a **Translation Target Language** chosen by the user for that request.
- A workspace member is enough to use **Workspace AI Chat**; document-specific actions additionally require readable access to attached documents.
- External document collaborators and public viewers cannot use **Workspace AI Chat**.
- The first AI assistance behavior is invoked through an **AI Chat Panel Action**, not a selection toolbar or document toolbar menu.
- The workspace-level AI shell hosts the **AI Chat Panel**.
- **AI Document Context Registration** connects the current document route to the workspace-level **AI Chat Panel** while the document screen is mounted, but the user may remove the resulting **AI Document Attachment**.
- The first **AI Chat API** streams **Streaming AI Response** content for chat turns.
- A **Streaming AI Response** consumes Subscription Plans **AI Response Gate** access only when it reaches a valid completed state and is persisted.
- Interrupted or canceled **Streaming AI Response** output releases or expires its Subscription Plans **AI Response Reservation**.
- Existing web component names may keep `AiChatPanel` as implementation names because the first release is chat-based.
- The **AI Chat Panel** shows selectable **AI Conversation Sessions**, optional **AI Document Attachments**, loading, assistant responses, **AI Transform Draft**, **AI Size Limit Notice**, Subscription Plans **AI Trial Counter**, Subscription Plans **Upgrade Prompt**, stale-draft state, apply, copy, and regenerate actions.
- An **AI Chat API** request identifies the workspace, chat message, current session, and any explicit **AI Document Attachments**; a future translate request will also include **Translation Target Language**.
- The first **AI Chat API** is workspace/session-scoped and supports explicit document attachments; it is not a workspace RAG endpoint.
- The **AI Chat API** rechecks workspace membership, document access for each **AI Document Attachment**, checks Subscription Plans **AI Response Gate**, resolves any required **AI Transform Source Snapshot**, enforces the direct-context size limit, records **AI Transform Metadata**, and returns an **AI Transform Draft** or assistant response.
- The **AI Chat API** uses an **AI Source Query** rather than the document detail read path, so generating a response does not record a document visit.
- **AI Chat Panel Action** calls the **AI Chat API**; the web client does not call the LLM provider directly.
- An **AI Append Action** appends the selected assistant response to the end of the currently active document; it does not insert at cursor position or replace the current selection.
- An **AI Append Action** is available only when the **AI Chat Panel** is hosted from an active editable document route; no active document or read-only access means no append action.
- **AI Append Block Conversion** converts assistant response text or Markdown-lite into basic document blocks such as paragraphs, bullets, numbered lists, and headings when supported; unsupported syntax degrades to plain text.
- An **AI Append Action** becomes available only after the assistant message is complete and persisted; partial **Streaming AI Response** text cannot be appended.
- The same assistant message can be appended more than once; each **AI Append Action** is a separate explicit user action.
- **AI Append Action** does not use **Stale AI Transform Draft** checks because it appends new content rather than replacing source content.
- **AI Append Action** targets the current active editable document, even if the assistant message originally used a different **AI Document Attachment** as source context.
- Removing an **AI Document Attachment** affects future AI context only; it does not hide **AI Append Action** while the user remains on an active editable document route.
- **AI Append Action** records AI-assisted provenance metadata with conversation session id, assistant message id, action type, actor, and timestamp; it does not store raw prompt or source snapshot text as document body/history.
- **AI Append Action** does not call the LLM provider and does not cause Subscription Plans **AI Response Consumption**.
- After a successful **AI Append Action**, the **AI Chat Panel** stays open and shows a success toast; append success does not force focus, navigation, or panel close.
- Failed **AI Append Action** shows a retryable failure toast and does not mutate the AI conversation session.
- **AI Transform Draft** carries an **AI Source Snapshot Token** so apply can detect a **Stale AI Transform Draft** without storing source text.
- A **Stale AI Transform Draft** cannot be applied; the user must run a new **AI Text Transform** against the current Document Body source.
- An AI request that exceeds the direct-context size limit produces an **AI Size Limit Notice** instead of chunking, retrieving, or partially transforming content.
- A document-aware AI request with no meaningful Document Body text produces an **AI Empty Content Notice** instead of calling the LLM.
- An invalid generated draft produces an **AI Draft Failure Notice**; Camille does not insert malformed blocks, silently downgrade to plain text, or repair output in the first release.
- Operational rate limits may produce an **AI Rate Limit Notice**; Subscription Plans **AI Response Gate** failures produce Entitlement Denials.
- Workspace AI Chat consumes the workspace-owned Workspace AI Trial Pool through Subscription Plans **AI Response Gate**.

## Flagged Ambiguities

- "AI writing assistant first" means **Workspace AI Chat** with optional explicit document attachments, not RAG, workspace search, or autonomous document actions.
- "AI current document" means the current route can attach the Document Title and Document Body as explicit context, but the user may remove that attachment.
- "AI session grouping" means recency grouping by session last activity, not grouping by attached document.
- "No document badge" means general workspace assistant chat with no document context, not workspace RAG.
- "AI session visibility" means user-private sessions inside a workspace, not shared workspace or document-visible sessions.
- "AI session title" means auto-title from the first successful chat turn, not manual-only naming or attached-document title.
- "Reset conversation" means return to the **New AI Chat Draft**, not clearing, deleting, or pre-creating a persisted **AI Conversation Session**.
- "AI preserves formatting" means basic document blocks for transform drafts, not full rich text, embeds, mentions, comments, or complex editor state.
- "AI first release" means workspace chat with attached-document summarize; translate is visible as a **Pending Translation Action**, while shorten, expand, rewrite, extract tasks, workspace Q&A, and actions are out of scope.
- "Translate document" is pending; when implemented, it means translate to an explicitly selected **Translation Target Language** and replace the transformed source content.
- "Large document AI" means show an **AI Size Limit Notice**; it does not mean chunking or RAG in the first release.
- "Append to this document" means **AI Append Action**: append the selected assistant response to the end of the current active document.
- "Append availability" means active editable document route only, not whether the current message used or still shows a document attachment.
- "Remove document badge" means remove the document from future **AI Chat Turns**, not rewrite prior turn context.
- "Append action copy" means "Append to this document", not "Insert into this document".
- "Append format" means Markdown-lite to basic document blocks, not one plain paragraph, full markdown import, or provider-returned document JSON.
- "Append during streaming" is rejected; append uses completed persisted assistant messages only.
- "Repeat append" means allowed duplicate appends, not disabled after first use.
- "Append provenance" means session id, assistant message id, action type, actor, and timestamp metadata, not full prompt/source retention.
- "Append billing" means no additional AI response consumption; billing happened when the assistant response was generated.
- "Append success feedback" means toast plus unchanged panel state, not closing the panel or auto-scrolling the document.
- "Append failure" means retryable toast without chat state mutation, not silent disable, blocking dialog, or clipboard fallback.
- "Summarize document" means produce a short bullet **Summary Shape** from an attached document, not a paragraph, exhaustive recap, or structured report by default.
- "AI draft changed underneath" means a **Stale AI Transform Draft** is blocked from applying, not that **AI Append Action** is blocked by document drift.
- "Streaming AI consumption" means completed persisted assistant response only, not stream start or partial text display.
- "AI response" means a **Streaming AI Response** for an **AI Chat Turn**, not a background job.
- "AiChatPanel naming" matches the chat-based first release.
- "AI implementation boundary" means a backend-owned **AI Chat API**, not a web-owned LLM call.
- "Current document for AI" means the server materialized projection at request time when the document is attached; unsynced browser-local edits are not included until collaboration sync updates the projection.
- "AI API request" means workspace id/session id plus chat message or transform intent from an **AI Chat Panel Action**, with explicit document attachments and target language for translate; source content is derived server-side.
- "AI chat panel" means workspace chat with optional attached document context first; workspace Q&A chat waits for a later RAG boundary.
- "AI response" means a generated assistant response for an **AI Chat Turn**, not streaming token events or background job state.
- "Empty document AI" means show an **AI Empty Content Notice** for document-aware actions; the Document Title alone is not transformed.
- "Bad AI output" means show an **AI Draft Failure Notice** and allow retry; it does not mean best-effort insertion.
- "AI rate limit" means operational cost and abuse control, separate from Subscription Plans **AI Response Gate** and **Upgrade Prompt**.
- "External viewer AI" is rejected; document-specific collaborators or public viewers who are not workspace members cannot use **Workspace AI Chat**.
- "Selected text AI" is out of scope for the first release; stable range identity and selection-toolbar UX are later decisions.
- "AI endpoint placement" means a workspace/session **AI Chat API** that supports explicit document attachments, not workspace RAG or a document-only transform endpoint.
- "AI source read" means a dedicated **AI Source Query**, not reusing `GetDocumentUseCase` or reading persistence entities directly from the use case.
- "Workspace AI shell" is a UI host for the **AI Chat Panel**, not a workspace-wide knowledge retrieval boundary.
- "AI panel document context" means route-local **AI Document Context Registration** creates a removable **AI Document Attachment**, not mandatory document scope.
