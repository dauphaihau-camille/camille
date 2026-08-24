# Context Map

Camille has multiple domain contexts. Each context owns its own glossary and relationships; architectural choices and tradeoffs live in [docs/adr/](./docs/adr/).

## Contexts

- [Document Editing](./docs/domain/document-editing/CONTEXT.md) - Changing document title, body, hierarchy, and access while collaborating in a workspace.
- [Subscription Plans](./docs/domain/subscription-plans/CONTEXT.md) - Workspace-owned plans, subscription lifecycle, entitlement gates, and seat-based billing.

## Relationships

- **Document Editing -> Subscription Plans**: Document editing asks Subscription Plans whether workspace growth is allowed when creating active content blocks.
- **Subscription Plans -> Document Editing**: Subscription Plans counts active document content blocks to evaluate workspace block entitlements.
