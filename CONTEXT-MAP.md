# Context Map

Camille has multiple domain contexts. Each context owns its own glossary and relationships; architectural choices and tradeoffs live in [docs/adr/](./docs/adr/).

## Contexts

- [Document Editing](./docs/domain/document-editing/CONTEXT.md) - Changing document title, body, hierarchy, and access while collaborating in a workspace.
- [AI Assistance](./docs/domain/ai-assistance/CONTEXT.md) - Workspace chat, explicit document attachments, AI-generated drafts, chat APIs, provider boundaries, and AI feature surfaces.
- [Subscription Plans](./docs/domain/subscription-plans/CONTEXT.md) - Workspace-owned plans, subscription lifecycle, entitlement gates, and seat-based billing.

## Relationships

- **Document Editing -> Subscription Plans**: Document editing asks Subscription Plans whether workspace growth is allowed when creating active content blocks.
- **Document Editing -> AI Assistance**: Document editing applies AI Assistance drafts as ordinary collaborative document edits when the user accepts them.
- **AI Assistance -> Document Editing**: AI Assistance uses explicit document attachments, document-scoped source snapshots, and route context to create chat responses and transform drafts.
- **AI Assistance -> Subscription Plans**: AI Assistance asks Subscription Plans whether workspace AI response access is available before generating an AI chat response or transform draft.
- **Subscription Plans -> Document Editing**: Subscription Plans counts active document content blocks to evaluate workspace block entitlements.
- **Subscription Plans -> AI Assistance**: Subscription Plans exposes AI response gate, trial counter, reservation, and upgrade state to AI Assistance.
