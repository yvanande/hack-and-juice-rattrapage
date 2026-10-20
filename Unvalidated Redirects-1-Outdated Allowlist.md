# Outdated Allowlist

## Methodology

1. Looked for how the application handles redirects to external resources (e.g. links to
   the project's social media, GitHub, or payment pages).
2. Used the browser's "Search" feature across all loaded source files (DevTools > Search,
   or Sources > global search) with the query `/redirect?to=` to list every URL
   whitelisted by the app's redirect endpoint.
3. Found several external URLs in `main.js`, including two cryptocurrency addresses:
   - a Bitcoin address, linked via `blockchain.info`
   - an Ethereum address, linked via `etherscan.io`
4. Tested the redirect endpoint directly in the browser address bar, requesting the
   Bitcoin address: `GET /redirect?to=https://blockchain.info/address/<address>`.
5. The browser was redirected to the blockchain.info page for that address, and the
   challenge was marked as solved immediately after the redirect.

Tools used: browser DevTools (global source search), manual URL testing in the address
bar.

## Vulnerabilities

- **Open redirect via outdated allowlist** — the `/redirect` endpoint only checks the
  target URL against a hardcoded list of allowed destinations. This list still contains
  an old cryptocurrency address that is no longer shown anywhere in the current storefront,
  meaning it was never cleaned up when the app's content changed.
  - Affected component: the redirect endpoint and its allowlist configuration.
  - Severity: low to medium — the entries are currently harmless, but the underlying
    mechanism is a maintenance/configuration flaw.

## Risks

- An allowlist that is not kept in sync with the application's actual content can end up
  authorizing destinations that are no longer under the organization's control (an
  expired domain, an abandoned payment address, etc.). If such a resource were ever taken
  over by a third party, the application would still happily redirect trusted users to
  it, which could be leveraged for phishing or scams while appearing to come from a
  trusted source (open redirect abuse).
- Business impact: reputational damage and potential financial fraud against users who
  trust a link because it originates from the company's own domain.

## Actions

- Regularly audit the redirect allowlist and remove entries that no longer correspond to
  content actually used or displayed by the application.
- Prefer validating redirect destinations against a dynamically maintained, versioned
  configuration rather than a hardcoded list buried in the client-side bundle.
- Consider adding a warning/interstitial page before redirecting users to external
  domains, so they can verify the destination themselves.