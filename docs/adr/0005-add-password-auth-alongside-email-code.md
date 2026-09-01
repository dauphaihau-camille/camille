# Add Password Auth Alongside Email-Code Login

Camille will add Password Login as an in-place auth-method switch on `/login`, keep Email-Code Login as the default login method, and change `/signup` to Password Signup with name, email, and password. Password reset will use `/forgot-password` for requesting a reset email and `/reset?t=<token>` for setting the new password, matching the API reset-link builder that already emits `/reset?t=` links.

The API already exposes password endpoints for register, login, forgot password, reset-token verification, and reset password. The web app should wire those contracts instead of creating new auth contracts. Successful password register, login, and reset should reuse the existing cookie-based auth response and post-login redirect behavior used by email-code and OAuth flows.

Password Reset must create or update the account's Password Credential. This is required because Email-Code Signup and OAuth Login can create Passwordless Accounts with no `user_credentials` row; resetting password for those accounts should establish the first credential rather than fail. The persistence implementation must therefore upsert the credential for a user instead of requiring one to already exist.

Forgot-password responses must remain enumeration-safe: requesting a reset for an unknown email returns the same accepted response as a known email, while only known accounts receive a reset email.
