# Login Jim

## Methodology

1. Opened the login form and attempted a classic SQL injection payload in the email
   field to see if the backend concatenates user input directly into a SQL query:
   `' OR 1=1--` (password field left with an arbitrary value).
2. This logged in successfully, but as the administrator account (the first user row in
   the database), not Jim specifically — confirming the injection works, but needs to be
   targeted.
3. Logged out and retried with an injection payload targeting Jim's email specifically:
   `jim@juice-sh.op' --`, with an arbitrary password.
4. This closes the quoted email string and comments out the rest of the original query
   (including the password check), effectively making the query equivalent to
   "WHERE email = 'jim@juice-sh.op'" with no password verification at all.
5. Login succeeded on Jim's account, confirmed via the Account page.

Tools used: browser only (login form), no additional tooling required.

## Vulnerabilities

- **SQL Injection in the login form** — the email field is concatenated directly into
  the backend SQL query without parameterization or input sanitization, allowing an
  attacker to alter the query's logic (e.g. bypass the password check entirely).
  - Affected component: authentication system / login endpoint.
  - Severity: critical — complete authentication bypass for any known or guessable
    email address.

## Risks

- An attacker can log into any user account (including administrators) without knowing
  their password, as long as the email address is known or guessable.
- Business impact: total compromise of account confidentiality and integrity; any user's
  personal data, orders, or privileges can be accessed by an attacker with no valid
  credentials at all.

## Actions

- Use parameterized queries / prepared statements for all database access involving user
  input, instead of building SQL strings by concatenation.
- Apply strict input validation and escaping as a defense-in-depth measure.
- Monitor and alert on repeated failed login attempts or suspicious login patterns (e.g.
  payloads containing SQL syntax) to detect injection attempts.