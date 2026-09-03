# Document Edit

Camille Document Edit describes the collaborative editing path from the browser Y.Doc through local persistence, cross-tab synchronization, Socket.IO, and PostgreSQL for both the Document Body and shared document metadata such as the Document Title.

The canonical collaborative state is Yjs. REST-readable document fields are derived projections used by search, publishing, navigation summaries, and existing document detail reads.

## Goals

- Let readable workspace members open a document with convergent collaborative title and body state.
- Let editable workspace members change the Document Title and Document Body through ordinary Yjs transactions.
- Preserve local-first editing through IndexedDB, BroadcastChannel, and state-vector reconnect synchronization.
- Persist server-accepted Yjs updates durably before broadcasting them to other sockets.
- Keep existing REST reads, search, publish, and navigation projections refreshed from canonical Yjs state.

## Scope

### In scope

- Browser document open and initial Y.Doc synchronization.
- Collaborative body editor mode for the Document Body.
- Shared `meta.title` inside the Y.Doc for the Document Title.
- Local persistence with `y-indexeddb`.
- Same-browser cross-tab update exchange with BroadcastChannel.
- Authenticated Socket.IO synchronization through the collaboration namespace.
- PostgreSQL Yjs update log, snapshots, sequence ordering, and SHA-256 deduplication.
- Server-side materialization of `documents.title`, `documents.content_json`, search text, publish reads, and subdocument references.
- Offline document editing for locally cached collaborative documents.
- Process-local operational boundary and the multi-instance pub/sub requirement.

### Out of scope

- Last-writer-wins JSON body replacement after collaborative state exists.
- Client-sent editor JSON as source of truth for collaborative documents.
- Durable undo or redo history across editing sessions.
- Persisted cursor, selection, presence, or awareness state.
- Cross-document transactionality for hierarchy, archive state, permissions, and sharing commands.
- Automatic conflict dialogs for Yjs updates; duplicate, delayed, and out-of-order updates converge through Yjs.
- Multi-instance Socket.IO fanout without a Redis adapter or equivalent pub/sub.

## Users

- Workspace members with readable access to a document can open the collaborative document route.
- Workspace members with edit permission can submit title and body Yjs updates.
- Workspace members with read-only access can receive synchronized state but cannot send accepted document edits.
- External public viewers and unauthenticated users are outside this collaborative editing flow unless a separate access model explicitly admits them.

## Documents

- [End-to-end flow](./flow.md)
- [Document open](./flows/document-open.md)
- [Local edit](./flows/local-edit.md)
- [Remote update](./flows/remote-update.md)
- [Offline and recovery](./flows/offline-and-recovery.md)
- [Canonical and derived state](./flows/canonical-and-derived-state.md)
- [Operational boundary](./flows/operational-boundary.md)
- [Undo and redo](./flows/undo-redo.md)
- [Normal edit undo and redo](./flows/undo-redo-normal-edit.md)
- [Undo and redo command compensation](./flows/undo-redo-command-compensation.md)
- [Archive subdocument undo and redo](./flows/archive-subdocument-undo-redo.md)
- [Duplicate subdocument undo and redo](./flows/duplicate-subdocument-undo-redo.md)
- [Undo and redo session boundaries](./flows/undo-redo-session-boundaries.md)

## Related Domain And Decision Docs

- [Document Editing Context](../../domain/document-editing/CONTEXT.md)
- [AI Assistance Context](../../domain/ai-assistance/CONTEXT.md)
- [Use Yjs-Level Session Undo For Document Editing](../../adr/0001-use-yjs-level-session-undo-for-document-editing.md)
- [Document Access Policy](../document-access-policy.md)
