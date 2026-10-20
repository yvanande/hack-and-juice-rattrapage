# Login Bender

## Methodology

1. Reused the exact SQL injection technique validated on the "Login Jim" challenge.
2. Guessed Bender's email address by following the same naming convention observed for
   Jim: `firstname@juice-sh.op`, giving `bender@juice-sh.op`.
3. Logged out of the previous session, then submitted the login form with email
   `bender@juice-sh.op' --` and an arbitrary password.
4. As with Jim's account, this comments out the password check in the backend query,
   authenticating as Bender with no valid password. Confirmed via the Account page.

Tools used: browser only (login form).

## Vulnerabilities

- Same root cause as "Login Jim": **SQL Injection in the login form**, allowing
  authentication bypass for any targeted email address.
  - Affected component: authentication system / login endpoint.
  - Severity: critical.

## Risks

- Same as "Login Jim": any account can be compromised without knowing its password, as
  long as the attacker can guess or find the associated email address (here, by
  following the predictable `firstname@juice-sh.op` pattern).
- Business impact: compounds the previous finding by showing the flaw is exploitable
  across multiple accounts using a predictable email naming convention, making
  large-scale account takeover realistic.

## Actions

- See "Login Jim" write-up: use parameterized queries, strict input validation, and
  monitoring for injection attempts.
- Additionally, avoid predictable email/username conventions for accounts where possible,
  or at least ensure the authentication mechanism itself is not reliant on secrecy of the
  email format as a security measure.