# Password Strength

## Methodology

1. Looked for the administrator's email address on the storefront. Found a partially
   masked email on the "About Us" page, in the form `***in@juice-sh.op`.
2. Deduced the full address based on the visible characters and Juice Shop's known
   conventions: `admin@juice-sh.op`.
3. Attempted to log in with `admin@juice-sh.op` and a guessed weak password,
   `admin123`, based on prior experience: the same password was previously observed on an
   administrator account during another pentest exercise (Epitech's Obsidian project).
4. The login succeeded on the first attempt, with no SQL injection or prior password
   change involved.

Tools used: browser only (no script or external tool needed), prior knowledge from a
previous pentest exercise.

## Vulnerabilities

- **Weak / guessable administrator password** — the admin account uses a common, weak
  password (`admin123`) with no enforced complexity requirements and no account lockout
  after failed attempts.
  - Affected component: authentication system (login form / user account for
    `admin@juice-sh.op`).
  - Severity: critical — direct compromise of the highest-privilege account in the
    application.

## Risks

- Full compromise of the administrator account grants access to every administrative
  feature of the application: user data, order data, configuration, and any sensitive
  business information exposed through the admin panel.
- Business impact: total loss of confidentiality and integrity for the application's
  data, severe reputational damage, and potential regulatory consequences if customer
  data is exposed.

## Actions

- Enforce a strong password policy for all accounts, especially privileged ones (minimum
  length, complexity requirements, rejection of common/breached passwords).
- Implement account lockout or rate limiting after a small number of failed login
  attempts to slow down brute-force or credential-guessing attacks.
- Require administrators to change default or initial passwords on first login, and
  periodically rotate privileged credentials.
- Consider adding multi-factor authentication (MFA) for administrative accounts.