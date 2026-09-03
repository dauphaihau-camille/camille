# Remote Update

This flow describes how the collaboration API accepts, persists, projects, and broadcasts a Yjs update received over Socket.IO.

```mermaid
sequenceDiagram
  participant Socket as Client Socket
  participant Gateway as Collaboration Gateway
  participant Service as Document Collaboration Service
  participant Doc as Active Server Y.Doc
  participant Store as PostgreSQL
  participant Projection as Derived Projections
  participant Room as Document Room

  Socket->>Gateway: Send Yjs update
  Gateway->>Gateway: Confirm socket joined document room
  Gateway->>Service: Accept update request
  Service->>Service: Recheck edit permission
  Service->>Doc: Apply update to active server Y.Doc
  Service->>Store: Append update with sequence and hash
  Service->>Projection: Materialize title, body JSON, search, publish, references
  Service-->>Gateway: Persisted update accepted
  Gateway-->>Room: Broadcast update to other sockets
```

Summary:

- gateway confirms the socket joined the target document room
- application service rechecks edit permission for each remote update
- accepted update is applied to the active server Y.Doc
- binary update is appended to PostgreSQL with document sequence and SHA-256 deduplication hash
- merged Y.Doc is materialized into derived projections for title and body JSON content
- `documents.title`, `documents.content_json`, search text, publish reads, and subdocument references refresh from the projection
- canonical server Y.Doc is not replaced by derived JSON content
- gateway broadcasts only after durable persistence succeeds
- every 100 updates, API stores a compact Yjs snapshot and removes covered updates
