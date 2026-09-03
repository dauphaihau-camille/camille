# Document Open

This flow describes how a document editing session creates and synchronizes the browser Y.Doc before the title and body editor bind to collaborative state.

```mermaid
sequenceDiagram
  actor Member as Workspace Member
  participant Route as Document Editing Session
  participant Local as y-indexeddb
  participant Channel as BroadcastChannel
  participant Socket as Socket.IO Provider
  participant API as Collaboration API
  participant Doc as Server Y.Doc
  participant Editor as Body Editor

  Member->>Route: Open document for editing
  Route->>Route: Create session-scoped Y.Doc
  Local-->>Route: Apply cached Yjs updates
  Route->>Channel: Connect document-scoped channel
  Route->>Socket: Connect with HTTP-only access cookie
  Socket->>API: Join document room
  API->>API: Verify workspace membership and access
  Route->>API: Send browser state vector
  API->>Doc: Load snapshot and update log or initialize from content_json
  API-->>Route: Server state vector and missing update
  Route->>API: Send update missing from server
  Route->>Editor: Bind meta.title and body fragment
```

Summary:

- browser creates one Y.Doc for the document editing session
- `y-indexeddb` applies locally persisted updates before or alongside remote sync
- BroadcastChannel connects tabs in the same browser profile for the same document
- Socket.IO connects to `/collaboration` with the HTTP-only access cookie
- API verifies workspace membership and document access before joining `document:{documentId}`
- client and server exchange state vectors to calculate missing updates
- server lazily initializes the first collaborative Y.Doc from existing `content_json`
- body editor does not receive separate initial content in collaboration mode
- existing JSON content is converted into the Y.Doc only during first server initialization
- server seeds `meta.title` from `documents.title` during first collaborative initialization
