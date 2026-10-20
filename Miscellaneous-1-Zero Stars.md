# Zero Stars

## Methodology

1. Opened the Customer Feedback form on the Juice Shop instance.
2. Inspected the rating component with the browser's developer tools (Elements tab) and
   identified it as an HTML `<input type="range">` element with `min="1"` and `max="5"`.
3. Tried editing the `min` attribute directly in the DOM and moving the slider to the far
   left. The slider visually showed 0, but the Angular form kept an internal value of 1,
   so the Submit button stayed tied to the old value and the form could not be sent this way.
4. Opened the Network tab and submitted a normal feedback (1 star) to see what request the
   app sends. No working "edit and resend" option was available in this browser, so the
   request was rebuilt manually instead.
5. Retrieved the authentication token from the browser storage with:
   `localStorage.getItem('token')`
6. Looked for the CAPTCHA endpoint. A first guess (`/api/Feedbacks/captcha`) returned a
   404. Reloading the feedback page and inspecting the Network tab revealed the real
   endpoint used by the app: `GET /rest/captcha/`.
7. Called that endpoint directly with `fetch()`, using the stored token for authentication.
   The response contained the CAPTCHA id, the question, and, unexpectedly, the expected
   answer in plain text: `{ captchaId: 2, captcha: '5*8-6', answer: '34' }`.
8. Sent a handcrafted `POST` request to `/api/Feedbacks` with `fetch()`, using:
   - `rating: 0` (below the UI's allowed minimum of 1)
   - the real `captchaId` and `captcha` answer obtained in the previous step
   - a negative comment, to match the "devastating" feedback requested by the challenge
9. The server accepted the request (HTTP 200) and the challenge was marked as solved.

Tools used: browser DevTools (Elements, Console, Network tabs), `fetch()` API calls from
the browser console.

## Vulnerabilities

- **Client-side-only input validation** — the rating is only constrained by the HTML
  `min`/`max` attributes and by Angular's form validation in the browser. The backend API
  (`/api/Feedbacks`) does not re-validate the `rating` value, so a value outside the
  intended 1–5 range is accepted.
  - Affected component: Feedback API (`/api/Feedbacks`), backend validation layer.
  - Severity: medium (data integrity issue, no direct access to other users' data).
- **CAPTCHA answer disclosure** — the `/rest/captcha/` endpoint returns the expected
  answer together with the question, making the CAPTCHA purely decorative.
  - Affected component: CAPTCHA generation endpoint (`/rest/captcha/`).
  - Severity: medium (defeats the anti-automation purpose of the CAPTCHA).

## Risks

- Arbitrary or out-of-range data can be injected into the application's feedback system,
  which could corrupt statistics, dashboards, or any feature relying on the rating field
  being within 1–5.
- Because the CAPTCHA can be solved automatically (the answer is sent by the server
  itself), an attacker can script mass submissions of feedback or abuse any other
  CAPTCHA-protected endpoint, leading to spam or denial-of-service-style abuse.
- Business impact: data integrity loss, reduced trust in displayed ratings/statistics, and
  exposure to automated abuse.

## Actions

- Re-validate all numeric inputs server-side (e.g., reject any `rating` outside 1–5),
  regardless of what the frontend allows.
- Never return the expected CAPTCHA answer (or any value that allows deriving it) in the
  API response; only return the question/challenge to solve.
- Add rate limiting and/or server-side logging on sensitive endpoints such as
  `/api/Feedbacks` and `/rest/captcha/` to detect and slow down automated abuse.
- Follow the general best practice of "never trust the client": treat all client-supplied
  data as untrusted until validated on the server.