# Separate Subscription Domain from Workspace Collaboration

Camille will model subscription plans in a dedicated Subscription domain instead of storing billing behavior directly in the Workspace domain. Workspace remains the collaboration boundary, while Subscription owns plan lifecycle, provider status mapping, checkout, webhooks, seat synchronization, and effective entitlement calculation; this keeps Stripe-specific state out of workspace collaboration logic and prevents entitlement checks from scattering provider details through product code.
