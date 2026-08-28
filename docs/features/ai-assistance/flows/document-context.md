# Document Context

This flow describes how the active document becomes explicit AI context and how source text is resolved server-side.

```mermaid
sequenceDiagram
  actor Member as Workspace Member
  participant Route as Document Route
  participant Panel as AI Chat Panel
  participant API as AI Chat API
  participant Access as Document Access
  participant Source as AI Source Query

  Route->>Panel: Register current document context
  Panel-->>Member: Show removable document attachment
  Member->>Panel: Submit document-aware action
  Panel->>API: Request with explicit document attachments
  API->>Access: Recheck workspace membership
  API->>Access: Recheck readable access for each attachment
  Access-->>API: Access allowed
  API->>Source: Resolve AI Transform Source Snapshot
  Source-->>API: Document title, body projection, snapshot token
  API-->>Panel: Continue generation or return notice
```

Summary:

- document context is explicit attachment context, not workspace retrieval
- user can remove the active document attachment before future turns
- earlier turns keep their original attachment records
- server resolves title and body through AI Source Query
- client-sent editor text is not source of truth
- unsynced local edits are excluded until collaboration sync updates projection
- empty body and size-limit cases return notices before provider calls
