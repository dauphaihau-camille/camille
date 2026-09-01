# Camille Auth

Camille Auth owns how a person proves identity, starts a session, and recovers credential access.

## Language

**Auth Method**:
A user-selectable way to prove identity during login or signup: email code, password, Google OAuth, or GitHub OAuth.
_Avoid_: Login type, provider button

**Email-Code Login**:
A passwordless login flow that sends a 6-digit code to an existing account email and verifies it before issuing a session.
_Avoid_: Magic link, OTP auth

**Email-Code Signup**:
A passwordless signup flow that sends a 6-digit code to a new account email and verifies it before creating the account and issuing a session.
_Avoid_: Email verification after signup, passwordless register

**Password Login**:
A login flow that verifies an email and password against a stored password credential before issuing a session.
_Avoid_: Manual login, normal login

**Password Signup**:
A signup flow that collects name, email, and password, creates a password credential, and issues a session.
_Avoid_: Register form, credential signup

**Password Credential**:
The stored password hash and update timestamp associated with a user account.
_Avoid_: Password, user password

**Passwordless Account**:
A user account created by email-code signup or OAuth that has no Password Credential.
_Avoid_: Social account, incomplete account

**Password Reset Request**:
A logged-out recovery request that accepts an email address, stores a reset token for known accounts, and returns the same accepted response for unknown accounts.
_Avoid_: Forgot form, account lookup

**Password Reset Token**:
A short-lived secret sent by email that authorizes setting a new Password Credential.
_Avoid_: Reset code, verification token

**Password Reset**:
The flow that verifies a Password Reset Token, creates or updates the account's Password Credential, revokes existing sessions, and issues a fresh session.
_Avoid_: Change password, recover account

**OAuth Login**:
A login or signup flow that authenticates through Google or GitHub, links an OAuth account, and issues a Camille session.
_Avoid_: Social login when the provider identity is not the domain concept

**Post-Login Redirect**:
The safe app-relative destination selected after any successful Auth Method issues a session.
_Avoid_: Callback URL, return URL

## Decisions

- `/login` should keep email-code login as the default surface and expose Password Login as an in-place Auth Method switch below GitHub.
- Password Login displays Email, Password, a forgot-password link, and a switch back to email-code login.
- `/signup` should require password: Name, Email, and Password call Password Signup.
- Password Reset must support Passwordless Accounts by creating the first Password Credential when none exists.
- The web reset request surface is `/forgot-password`; emailed reset links land on `/reset?t=<token>`.
- Forgot-password responses must not disclose whether an email belongs to an account.
