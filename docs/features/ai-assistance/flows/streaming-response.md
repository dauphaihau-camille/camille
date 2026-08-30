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
  Provider-->>API: Stream raw response chunks
  API-->>Panel: Stream block_start/text_delta/block_end events
  Panel-->>Member: Render partial assistant block preview
  Provider-->>API: Completed output
  API->>Store: Persist valid turn, assistant text, and block payload
  API->>Billing: Consume reservation after persistence
  API-->>Panel: Completed persisted assistant message with reconciled block payload
```

Summary:

- streaming is response delivery for an AI Chat Turn, not a background job
- partial streaming block previews are visible but cannot be appended
- append uses the completed persisted response block payload; copy uses completed assistant text
- invalid generated output shows a retryable failure notice
- failed, canceled, or interrupted streams do not consume trial access unless a valid response was persisted
