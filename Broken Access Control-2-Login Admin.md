# Login Admin

## Methodology

1. Same action as the "Password Strength" challenge: logged into the application using
   the administrator's email (`admin@juice-sh.op`), found on the "About Us" page, and a
   guessed weak password (`admin123`), reused from a previous pentest exercise
   (Epitech's Obsidian project).
2. The login succeeded, granting access to the administrator account, which solved this
   challenge alongside "Password Strength".

Tools used: browser only.

## Vulnerabilities

- **Broken access control via compromised administrator credentials** — gaining the
  admin account's credentials provides full access to every administrative feature and
  privileged action in the application, with no additional verification step.
  - Affected component: authentication system and the administrative features it
    unlocks.
  - Severity: critical.

## Risks

- Same root cause as "Password Strength": a weak administrator password directly
  translates into a full access control bypass, since the application relies solely on
  the login credentials to grant admin-level privileges.
- Business impact: an attacker with admin access can view, modify, or delete any data in
  the system, including other users' personal information and orders.

## Actions

- See "Password Strength" write-up: enforce strong password policies, account lockout,
  and MFA for privileged accounts.
- Additionally, consider privilege separation or step-up authentication (e.g. requiring
  re-authentication) for the most sensitive administrative actions, so that a single
  compromised password does not grant unrestricted access to everything at once.