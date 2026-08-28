# Streaming Response

This flow describes how an AI Chat Turn streams to the panel and becomes persisted product state.

```mermaid
sequenceDiagram
  actor Member as Workspace Member
  participant Panel as AI Chat Panel
  participant API as AI Chat API
  participant Provider as LLM Provider
  participant Store as Conversation Store
  participant Billing as Subscription Plans

  Member->>Panel: Submit chat or summarize action
  Panel->>API: Start AI Chat Turn
  API->>Provider: Generate response
  Provider-->>API: Stream response chunks
  API-->>Panel: Streaming AI Response
  Panel-->>Member: Render partial assistant text
  Provider-->>API: Completed output
  API->>Store: Persist valid completed turn
  API->>Billing: Consume reservation after persistence
  API-->>Panel: Completed persisted assistant message
```

Summary:

- streaming is response delivery for an AI Chat Turn, not a background job
- partial streaming text is visible but cannot be appended
- append and copy actions use completed persisted assistant messages
- invalid generated output shows a retryable failure notice
- failed, canceled, or interrupted streams do not consume trial access unless a valid response was persisted
