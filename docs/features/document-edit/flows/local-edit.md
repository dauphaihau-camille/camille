# Local Edit

This flow describes how title and body edits become local Yjs updates and move through browser-local persistence, cross-tab synchronization, and the Socket.IO provider.

```mermaid
sequenceDiagram
  actor Member as Workspace Member
  participant Title as Title Editing Surface
  participant Body as Body Editor
  participant Doc as Y.Doc
  participant Local as y-indexeddb
  participant Channel as BroadcastChannel
  participant Socket as Socket.IO Provider
  participant Tab as Other Local Tab

  Member->>Title: Edit document title
  Title->>Doc: Write meta.title
  Member->>Body: Edit document body
  Body->>Doc: Write collaborative Y.XmlFragment
  Doc-->>Local: Persist binary update
  Doc-->>Channel: Publish update with local origin
  Channel-->>Tab: Apply update in sibling tab
  Doc-->>Socket: Send update when connected
```

Summary:

- title edits write to `meta.title`
- body edits write to the collaborative body fragment
- every title or body change emits a binary Yjs update
- `y-indexeddb` persists updates for recovery and offline use
- BroadcastChannel sends updates to other local tabs for the same document
- Socket.IO provider sends updates to the API when connected
- updates received from BroadcastChannel or Socket.IO use transport-specific origins
- transport-specific origins prevent echo loops back to the origin transport
- Yjs updates are commutative and idempotent, so duplicate, delayed, and out-of-order delivery converges
