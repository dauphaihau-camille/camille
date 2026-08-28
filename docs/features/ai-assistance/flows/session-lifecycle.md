# Session Lifecycle

This flow describes how the AI Chat Panel moves from local draft state to a persisted user-private conversation session.

```mermaid
sequenceDiagram
  actor Member as Workspace Member
  participant Panel as AI Chat Panel
  participant API as AI Chat API
  participant Store as Conversation Store

  Member->>Panel: Open AI chat
  Panel-->>Member: Show New AI Chat Draft
  Member->>Panel: Submit first chat action
  Panel->>API: Create turn with optional session_id
  API->>Store: Create or resolve AI Conversation Session
  API->>Store: Persist user message and completed assistant response
  Store-->>API: Conversation session and turn
  API-->>Panel: Completed response and session state
  Panel-->>Member: Show titled session in recency group
  Member->>Panel: New AI Conversation Action
  Panel-->>Member: Return to New AI Chat Draft
```

Summary:

- New AI Chat Draft is local, not a persisted empty session
- first submitted turn creates or resolves the AI Conversation Session
- AI Conversation Sessions are user-private inside a workspace
- successful first response titles the session
- session grouping follows last activity, not attached document
- New AI Conversation Action does not delete existing sessions
