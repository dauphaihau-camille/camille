# Camille Subscription Plans

Camille subscription plans control paid access to workspace-level collaboration and document limits.

## Language

**Workspace Subscription**:
A workspace-owned billing relationship that determines the workspace's plan, subscription status, and paid seat count.
_Avoid_: User subscription, account subscription

**Subscription Status**:
The application-level lifecycle state that determines which entitlements currently apply.
_Avoid_: Provider status, payment status

**Subscription Summary**:
The workspace-facing view of current plan, subscription status, seat count, and effective entitlements.
_Avoid_: Provider record, invoice

**Checkout Session**:
A short-lived payment flow used to start a paid plan for a workspace.
_Avoid_: Subscription activation, payment proof

**Billing Webhook**:
A verified payment-provider event that updates the workspace subscription lifecycle in Camille.
_Avoid_: Checkout success, browser redirect

**Billing Tab**:
The workspace settings surface for plan state and subscription actions.
_Avoid_: Pricing page, invoice portal

**Upgrade Prompt**:
A concise product message shown when an entitlement gate denies growth and the workspace can resolve it by upgrading.
_Avoid_: Billing page, editor billing controls

**Entitlement Denial**:
An API response that rejects a product action because the workspace's effective entitlements do not allow it.
_Avoid_: Payment error, validation error

**Plan**:
A named product tier that defines baseline workspace entitlements.
_Avoid_: Package, pricing level

**Free Plan**:
The default free workspace plan with conditional block limits based on workspace member count.
_Avoid_: Trial, unpaid subscription

**Plus Plan**:
The paid monthly workspace plan billed per seat with unlimited blocks.
_Avoid_: Pro, business plan

**Past Due Subscription**:
A paid workspace subscription whose payment has failed while paid entitlements still apply temporarily.
_Avoid_: Failed plan, disabled subscription

**Canceling Subscription**:
A paid workspace subscription scheduled to return to the Free Plan at the end of the current billing period.
_Avoid_: Canceled subscription, deleted subscription

**Downgraded Workspace**:
A workspace that moved from Plus Plan to Free Plan after paid access ended.
_Avoid_: Deleted workspace, suspended workspace

**Entitlement**:
A rule that determines whether a workspace can use a feature or exceed a product limit.
_Avoid_: Permission, role

**Effective Entitlement**:
The entitlement result after combining the workspace's plan with current workspace state.
_Avoid_: Static plan limit, hardcoded plan check

**Block Limit**:
The maximum number of active content blocks a workspace may contain before block creation is denied.
_Avoid_: Document limit, page limit

**Workspace Block Count**:
The total number of active BlockNote content blocks across all non-archived documents in a workspace, including nested child blocks.
_Avoid_: Document count, page count

**Block Creation Gate**:
The product check that denies net-new active content blocks when the workspace has reached its effective Block Limit.
_Avoid_: Member limit, document permission

**Over-Limit Workspace**:
A workspace whose current block count is greater than its effective Block Limit.
_Avoid_: Locked workspace, suspended workspace

**Seat Count**:
The number of active workspace members counted for per-member billing.
_Avoid_: User count, member limit

**Seat Sync**:
The process that updates paid subscription quantity to match the workspace Seat Count.
_Avoid_: Local invoice calculation, member permission update

**Billable Member**:
An active workspace owner, admin, or member included in seat-based billing.
_Avoid_: Guest, viewer, collaborator

## Relationships

- A **Workspace** owns at most one active **Workspace Subscription**.
- A **Workspace Subscription** selects exactly one **Plan** and has one **Subscription Status**.
- **Plan** definitions are product-level tiers; **Effective Entitlements** are computed from the selected **Plan** and workspace state.
- **Billing Webhook** events are the source of truth for paid subscription lifecycle changes.
- **Checkout Session** starts payment but does not activate paid access by itself.
- A **Billing Tab** is the subscription management home; an **Upgrade Prompt** appears only at contextual entitlement gates.
- A **Block Creation Gate** failure returns an **Entitlement Denial** with enough context for clients to explain the limit.
- The initial plan set is **Free Plan** and monthly **Plus Plan**.
- A free **Subscription Status** applies **Free Plan** entitlements; active, **Past Due Subscription**, and **Canceling Subscription** states apply **Plus Plan** entitlements while paid access remains effective.
- A **Canceling Subscription** becomes free when its paid period ends.
- Product authorization and entitlement checks use **Subscription Status**, not raw provider state.
- A **Free Plan** workspace with one **Billable Member** has an unlimited **Block Limit**.
- A **Free Plan** workspace with two or more **Billable Members** has a 1,000 **Block Limit**.
- A **Plus Plan** workspace has an unlimited **Block Limit**.
- **Workspace Block Count** includes active BlockNote blocks such as paragraphs, headings, to-dos, subdocument blocks, toggles, and nested children.
- **Workspace Block Count** excludes archived documents, document titles, teamspaces, comments, favorites, visits, and preferences.
- **Block Creation Gate** applies to root document creation, subdocument creation, REST content replacement, and collaborative editor updates that add net-new blocks.
- **Block Creation Gate** compares net-new block growth, so edits that only modify, reorder, or remove existing blocks are not denied by the block limit.
- Collaborative editor clients may optimistically apply local edits, but changes that increase the block count beyond the effective **Block Limit** are reverted while existing-block text edits remain allowed.
- The server remains the source of truth for collaborative editor block growth; client-side rollback is a product feedback mechanism, not the authoritative entitlement check.
- An **Over-Limit Workspace** remains readable and editable for existing content, but cannot create additional active content blocks.
- A **Downgraded Workspace** keeps existing content and becomes an **Over-Limit Workspace** if its **Workspace Block Count** exceeds the Free Plan limit.
- The initial subscription model does not cap workspace member count.
- **Seat Count** is derived from **Billable Members**, not document-specific collaborators or public viewers.
- **Seat Sync** follows **Seat Count** changes for paid workspaces.
- Camille does not calculate invoices locally in the initial subscription model.

## Flagged Ambiguities

- "Subscription" means **Workspace Subscription**, not a user-level subscription.
- "Plan" means the initial **Free Plan** and monthly **Plus Plan**, not a full pricing catalog.
- "Free block limit" means a conditional **Effective Entitlement**, not a static limit on the **Free Plan**.
- "Member count" means **Seat Count** for billing and entitlement rules; document-specific collaborators and public viewers do not count.
- "Canceled" is split into **Canceling Subscription** before paid access ends and free status after the billing period ends.
- "Checkout success" is not payment proof; **Billing Webhook** events update subscription lifecycle.
- "Free member limit" is rejected; Free collaboration is limited through the **Block Creation Gate**.
- "Block count" means **Workspace Block Count**, not document count, title count, or total workspace records.
- "Over limit" means blocking new content growth, not locking existing workspace access.
