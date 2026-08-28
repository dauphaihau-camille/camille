# Append To Document

This flow describes how a completed assistant response becomes an explicit AI-Assisted Document Edit in the current active document.

```mermaid
sequenceDiagram
  actor Member as Workspace Member
  participant Panel as AI Chat Panel
  participant Converter as Block Converter
  participant Editor as Document Editor
  participant Yjs as Collaborative Y.Doc
  participant Toast as Feedback Toast

  Member->>Panel: Click Append to this document
  Panel->>Converter: Convert assistant response Markdown-lite
  Converter-->>Panel: Supported basic document blocks
  Panel->>Editor: Append blocks to current active document
  Editor->>Yjs: Write ordinary collaborative document edit
  Yjs-->>Editor: Synchronized AI-Assisted Document Edit
  Editor-->>Panel: Append result with provenance metadata
  Panel->>Toast: Show success or retryable failure
```

Summary:

- append requires an active editable document route
- append targets the current active document, not necessarily the original attachment
- append means end-of-document append, not cursor insertion or selected-range replacement
- only completed persisted assistant messages can be appended
- unsupported Markdown-lite degrades to plain text
- repeated append creates another explicit Document Edit
- append does not call the LLM provider or consume another AI response
- provenance stores session id, assistant message id, action type, actor, and timestamp
