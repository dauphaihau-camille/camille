# Billing And Retention

This flow describes AI response gating, trial-pool consumption, and retained AI data.

```mermaid
sequenceDiagram
  actor Member as Workspace Member
  participant API as AI Chat API
  participant Billing as Subscription Plans
  participant Provider as LLM Provider
  participant Store as Conversation Store
  participant Metadata as AI Transform Metadata

  Member->>API: Submit AI Chat Panel Action
  API->>Billing: Check AI Response Gate
  Billing-->>API: Reserve trial response or allow Business access
  API->>Provider: Generate response
  Provider-->>API: Completed valid output
  API->>Store: Persist chat turn and assistant response
  API->>Billing: Consume reservation after persistence
  API->>Metadata: Store operational metadata only
  API-->>Member: Completed response
```

Summary:

- Free Plan and Plus Plan use a workspace-owned shared trial pool
- initial trial allowance is 10 plus 5 per Billable Member
- AI Response Gate reserves before provider generation
- valid completed persisted responses consume one reservation
- empty, size-limit, entitlement-denied, rate-limited, failed, invalid, canceled, and interrupted outcomes do not consume trial access
- leaked reservations expire after 10 minutes
- Business Plan bypasses trial-pool consumption for ongoing AI access
- chat messages are product state; raw source snapshots and unapplied draft text are not retained as hidden history
