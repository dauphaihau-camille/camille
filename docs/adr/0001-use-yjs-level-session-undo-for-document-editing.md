# Use Yjs-Level Session Undo For Document Editing

## Status

Accepted

## Context

Camille document editing stores collaborative body state in a Y.Doc and now stores document title as shared Yjs metadata. Users expect Notion-like undo/redo while editing: title edits, body edits, formatting, block insertion/removal, and visible placement effects from document commands should share one recency-ordered editing-session history.

Undo/redo must stay local to the current browser editing session. It must not become durable page history, a global collaborator rollback, or persisted command history. Remote collaborator updates, socket replay, broadcast replay, initialization, and system synchronization must not become local undo entries.

Some document commands have backend side effects as well as visible body effects. Archiving a subdocument block removes the visible block and archives the child document. Duplicating a subdocument block can create a new subtree and visible placement. Undoing those visible effects without compensating backend state would leave broken or hidden active documents.

## Decision

Use one browser-local Yjs-level Session Undo Manager scoped to the collaborative Y.Doc for the current document screen.

The Session Undo Manager tracks local user edit origins for title and body changes, including BlockNote/y-prosemirror body transactions. It excludes remote, socket, broadcast, initialization, and system synchronization origins. Keyboard shortcuts invoke undo/redo only when focus is in the title input or body editor and only while the document is editable.

Document commands are undoable only for their visible placement effect in the current document body. Command-related undo entries will keep lightweight in-memory Command Undo Metadata beside the session history entry when backend compensation is required. Compensating Commands must validate current permissions, target identity, document relationship, and lifecycle state before replaying the visible effect.

## Consequences

- Title and body edits share one recency-ordered undo/redo stack.
- Continuous editor typing can use Yjs/BlockNote capture windows rather than a custom character history.
- Undo/redo history clears on navigation, reload, collaboration readiness loss, permission loss, or archived state.
- Undo/redo results synchronize as ordinary collaborative Yjs updates, while another user's updates do not enter this user's local stack.
- Command compensation remains an explicit command contract instead of being hidden inside local Yjs replay.
- Undo/redo does not restore history across pages, tabs, browser restarts, or old document versions.

## Alternatives Considered

### Separate title and body histories

Rejected. It would make undo order depend on focused surface rather than actual edit recency, which conflicts with Notion-like behavior.

### ProseMirror body history plus custom title history

Rejected. It would duplicate history semantics around the collaborative Y.Doc and would not naturally include title/body ordering or command placement effects.

### Persisted command history

Rejected. The requested behavior is browser-local editing-session undo, not durable page history or audit-log rollback.

### Backend-first global page history

Rejected. It would behave across navigation and collaborators in ways that do not match the product rule: undo/redo should reverse current-document visible block-level effects, not global page history.
