# View Basket

## Methodology

1. Added an item to the shopping basket to trigger a basket-related request, and
   inspected the Network tab to find the relevant API call.
2. Identified the endpoint used by the application to read basket contents:
   `GET /rest/basket/{basketId}`. Found via the frontend source code
   (`main.js`), which builds this URL as `${this.hostServer}/rest/basket/${e}`.
3. Confirmed, via a `BasketItems` API response, that the authenticated account's own
   basket id is `6`.
4. Sent a manual request to a different basket id, `1`, using the current account's own
   authentication token:
```js
   fetch('https://ctf.juice.cyber.epitest.eu/rest/basket/1', {
     headers: { 'Authorization': 'Bearer ' + localStorage.getItem('token') }
   }).then(r => r.json()).then(console.log);
```
5. The request succeeded (`status: 'success'`) and returned the contents of basket `1`,
   which belongs to a different account than the one used to authenticate. This confirms
   the application does not verify that the requested basket belongs to the
   authenticated user.

Tools used: browser DevTools (Network and Sources tabs), manual `fetch()` call from the
console.

## Vulnerabilities

- **Broken Object Level Authorization (IDOR) on the basket endpoint** — the
  `/rest/basket/{id}` endpoint only checks that the requester is authenticated, not that
  the requested basket actually belongs to them. Any valid user can read any other user's
  basket by simply changing the numeric id in the request.
  - Affected component: basket API (`/rest/basket/{id}`).
  - Severity: high — direct unauthorized access to another user's data.

## Risks

- Any authenticated user can enumerate basket ids and read the shopping basket contents
  of other customers, exposing what products they are interested in or have added to
  their cart.
- Depending on what the basket contains or links to (pricing, personal notes, coupon
  codes applied), this could expose further sensitive information or enable further
  attacks (e.g. manipulating someone else's basket, not just reading it).
- Business impact: privacy violation for customers, potential competitive or reputational
  harm if basket data reveals business-sensitive information (e.g. B2B pricing or bulk
  orders).

## Actions

- Enforce object-level authorization on every endpoint that takes a resource id as a
  parameter: verify server-side that the authenticated user is the legitimate owner of
  the requested basket before returning any data.
- Avoid using simple sequential, guessable numeric ids for sensitive resources; prefer
  non-guessable identifiers (e.g. UUIDs) as an additional (but not sufficient on its own)
  layer of protection.
- Add automated tests and/or a dedicated security review step specifically covering
  Insecure Direct Object Reference (IDOR) patterns across all REST endpoints.