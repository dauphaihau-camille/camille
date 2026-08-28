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

**AI Trial Counter**:
A workspace-facing display of remaining complimentary AI responses.
_Avoid_: Personal AI balance, token meter

**Entitlement Denial**:
An API response that rejects a product action because the workspace's effective entitlements do not allow it.
_Avoid_: Payment error, validation error

**AI Response Gate**:
The entitlement check that denies AI feature use when a workspace has no remaining AI access.
_Avoid_: Rate limit, provider error, document permission

**Workspace AI Trial Pool**:
A workspace-owned pool of complimentary AI responses available before an AI-capable paid plan is required.
_Avoid_: User AI quota, monthly reset, per-account trial

**AI Response Reservation**:
An atomic hold on one available AI response while generation is in progress.
_Avoid_: Best-effort count, optimistic overage

**AI Response Consumption**:
The counting of one valid AI-generated response against the Workspace AI Trial Pool.
_Avoid_: Prompt count, token count, request attempt

**AI Trial Pool Counters**:
The granted, reserved, and consumed response counts used to enforce a Workspace AI Trial Pool.
_Avoid_: Document AI metadata, provider usage log

**AI Response Reservation Expiry**:
A 10-minute window after which an uncompleted AI Response Reservation stops counting against available trial responses.
_Avoid_: Permanent hold, support-only cleanup

**AI Trial Allowance**:
The total number of complimentary AI responses assigned to a Workspace AI Trial Pool.
_Avoid_: Per-user quota, monthly allowance

**Granted AI Trial Allowance**:
The cumulative complimentary AI responses granted to a workspace over time.
_Avoid_: Current-seat quota, reversible allowance

**Plan**:
A named product tier that defines baseline workspace entitlements.
_Avoid_: Package, pricing level

**Free Plan**:
The default free workspace plan with conditional block limits based on workspace member count.
_Avoid_: Trial, unpaid subscription

**Plus Plan**:
The paid monthly workspace plan billed per seat with unlimited blocks.
_Avoid_: Pro, business plan

**Business Plan**:
The paid $20-per-seat monthly workspace plan with ongoing AI responses.
_Avoid_: Plus Plan AI, AI add-on

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

**AI-Capable Plan**:
A plan whose effective entitlements allow ongoing AI responses after the Workspace AI Trial Pool is exhausted.
_Avoid_: AI trial, unlimited free AI

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
- An **AI Response Gate** failure returns an **Entitlement Denial** with enough context for clients to show an **Upgrade Prompt**.
- The initial plan set is **Free Plan**, monthly **Plus Plan**, and monthly **Business Plan**.
- A free **Subscription Status** applies **Free Plan** entitlements; active, **Past Due Subscription**, and **Canceling Subscription** states apply the workspace's selected paid plan entitlements while paid access remains effective.
- A **Canceling Subscription** becomes free when its paid period ends.
- Product authorization and entitlement checks use **Subscription Status**, not raw provider state.
- A **Free Plan** workspace with one **Billable Member** has an unlimited **Block Limit**.
- A **Free Plan** workspace with two or more **Billable Members** has a 1,000 **Block Limit**.
- A **Plus Plan** workspace has an unlimited **Block Limit**.
- **Workspace Block Count** includes active BlockNote blocks such as paragraphs, headings, to-dos, subdocument blocks, toggles, and nested children.
- **Business Plan** is the first **AI-Capable Plan**.
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
- **AI Response Consumption** happens when an AI action successfully produces a valid response; retrying generation consumes another response if it succeeds.
- **Free Plan** and **Plus Plan** use the **Workspace AI Trial Pool** for AI access until it is exhausted.
- Exhausting the **Workspace AI Trial Pool** requires **Business Plan** or another **AI-Capable Plan** for continued AI use.
- **AI Trial Allowance** is `10 + (5 × Billable Member count)` shared by the workspace.
- **Granted AI Trial Allowance** can increase when **Billable Member** count grows, but it does not decrease when members are removed.
- Increasing **Billable Member** count can increase the shared **Granted AI Trial Allowance** by 5 responses per added **Billable Member**, but does not create per-user AI quotas.
- The **AI Response Gate** creates an **AI Response Reservation** before the LLM provider is called.
- A successful valid AI response converts its **AI Response Reservation** into **AI Response Consumption** only after the response reaches a completed persisted state.
- Empty-content, size-limit, entitlement-denial, and rate-limit outcomes do not create an **AI Response Reservation**.
- Provider-failure and invalid-draft outcomes release their **AI Response Reservation**.
- Interrupted or canceled streaming responses release or expire their **AI Response Reservation** and do not cause **AI Response Consumption**.
- Reusing a completed AI response through AI Assistance **AI Append Action** does not create another **AI Response Reservation** or **AI Response Consumption**.
- A **Workspace AI Trial Pool** is one-time complimentary usage and does not reset over time.
- **Business Plan** bypasses **Workspace AI Trial Pool** consumption for ongoing AI access.
- A workspace downgraded from **Business Plan** to **Free Plan** or **Plus Plan** resumes its existing **Workspace AI Trial Pool** state.
- Empty-content, size-limit, entitlement-denial, rate-limit, provider-failure, and invalid-draft outcomes do not cause **AI Response Consumption**.
- Operational AI rate limits are separate from the **AI Response Gate** and do not produce Subscription Plans **Entitlement Denials**.
- An **AI Trial Counter** is shown in AI feature surfaces for **Free Plan** and **Plus Plan** workspaces while trial access applies.
- The **AI Trial Counter** displays remaining responses, not used-over-total progress by default.
- When the **Workspace AI Trial Pool** is exhausted, AI feature surfaces show an inline **Upgrade Prompt** that explains the shared workspace trial is used up.
- **Upgrade Prompt** gives workspace owners and admins an upgrade action; other workspace members see ask-admin copy.
- **Business Plan** workspaces do not need an **AI Trial Counter** for the Workspace AI Trial Pool.
- **Business Plan** is billed monthly at $20 per **Billable Member**.
- **Business Plan** AI access has operational rate limits only in the first release, not a visible monthly AI quota.
- **AI Response Gate** grants trial-pool AI access only to workspace members, not document-specific collaborators or public viewers.
- **AI Trial Pool Counters** are owned by Subscription Plans.
- AI Assistance asks the **AI Response Gate** and does not write **AI Trial Pool Counters** directly.
- An **AI Response Reservation** has a short **AI Response Reservation Expiry** so leaked in-progress generations do not lock the trial pool.
- The first **AI Response Reservation Expiry** is 10 minutes.

## Flagged Ambiguities

- "Subscription" means **Workspace Subscription**, not a user-level subscription.
- "Plan" means the initial **Free Plan**, monthly **Plus Plan**, and monthly **Business Plan**, not a full pricing catalog.
- "Free block limit" means a conditional **Effective Entitlement**, not a static limit on the **Free Plan**.
- "Member count" means **Seat Count** for billing and entitlement rules; document-specific collaborators and public viewers do not count.
- "Canceled" is split into **Canceling Subscription** before paid access ends and free status after the billing period ends.
- "Checkout success" is not payment proof; **Billing Webhook** events update subscription lifecycle.
- "Free member limit" is rejected; Free collaboration is limited through the **Block Creation Gate**.
- "Block count" means **Workspace Block Count**, not document count, title count, or total workspace records.
- "Over limit" means blocking new content growth, not locking existing workspace access.
- "AI count" means **AI Response Consumption** per valid generated response, not per prompt attempt, token, blocked request, or failed generation.
- "AI upgrade" means the workspace needs an **AI-Capable Plan** after the trial pool is exhausted.
- "Plus AI" is rejected; **Plus Plan** keeps trial-only AI access through the **Workspace AI Trial Pool**.
- "AI trial seat changes" means grant additional shared allowance for added billable members, not subtract allowance when members leave.
- "Five members using AI" means five users consume the same shared **Workspace AI Trial Pool**; seat count may increase the pool size but does not create five separate pools.
- "AI trial reset" is rejected; the **Workspace AI Trial Pool** is not a monthly quota.
- "Business AI usage" does not consume the **Workspace AI Trial Pool**.
- "Business downgrade AI" means resume the existing trial pool state, not grant a new AI trial.
- "AI responses remaining" means the **AI Trial Counter** for the shared workspace pool, not a personal counter.
- "AI user eligibility" means workspace members only; readable external collaborators do not consume or use the workspace AI pool.
- "Last AI response race" means reserve before provider call; Camille does not allow intentional trial-pool overage.
- "AI trial allowance formula" means 10 base responses plus 5 responses per **Billable Member**.
- "AI counter display" means remaining responses, for example `15 AI responses remaining`.
- "AI exhausted UX" means inline **Upgrade Prompt** in the AI feature surface, not toast-only feedback or automatic billing redirect.
- "AI upgrade prompt actor" means owners/admins can act, regular members get ask-admin copy.
- "Business pricing" means $20 per **Billable Member** per month.
- "Business AI limit" means operational abuse and cost controls only, not a monthly Business AI allowance.
- "AI trial storage" means Subscription-owned **AI Trial Pool Counters**, not AI Assistance metadata, Document Editing metadata, or provider logs.
- "AI reservation leak" means expired reservations stop counting against remaining responses; Camille does not consume trial responses for uncertain failures.
- "AI reservation TTL" means 10 minutes for the first release.
- "Streaming AI count" means completed persisted response only, not stream start or partial text display.
- "AI append billing" means append is a document edit with no additional AI response consumption.
