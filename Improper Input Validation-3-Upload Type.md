# Upload Type

## Methodology

1. Opened the "Complaint" form, which includes an "Invoice" file upload field.
2. Attempted to select a `.txt` file through the normal file picker and submit the form.
   The frontend immediately rejected it with "Forbidden file type. Only PDF, ZIP
   allowed." and kept the Submit button disabled, preventing any request from being sent.
3. To identify the real upload mechanism, submitted a valid request with a dummy `.pdf`
   file through the UI and inspected the Network tab. This revealed two separate
   requests: one to `/api/Complaints` (the complaint message) and a distinct one to
   `POST /file-upload` (the actual file), using a `file` field in a `multipart/form-data`
   body.
4. Rebuilt the file upload request manually from the browser console, bypassing the
   frontend's file-type check entirely:
```js
   const blob = new Blob(['ceci est un test'], { type: 'text/plain' });
   const file = new File([blob], 'test.txt', { type: 'text/plain' });
   const formData = new FormData();
   formData.append('file', file);

   fetch('https://ctf.juice.cyber.epitest.eu/file-upload', {
     method: 'POST',
     headers: { 'Authorization': 'Bearer ' + localStorage.getItem('token') },
     body: formData
   });
```
5. The server responded with `204 No Content`, confirming the `.txt` file was accepted,
   and the challenge was marked as solved.

Tools used: browser DevTools (Network tab, Console), manual `fetch()` call with a
`FormData` payload.

## Vulnerabilities

- **Client-side-only file type validation** — the allowed file extensions (`.pdf`,
  `.zip`) are only enforced by frontend JavaScript before the request is even built. The
  backend endpoint (`/file-upload`) does not re-validate the file's type/extension at
  all, accepting any file sent directly to it.
  - Affected component: file upload endpoint (`/file-upload`), used by the Complaint
    feature.
  - Severity: medium to high, depending on what the uploaded files are later used for
    (e.g. served back to other users, processed by another system).

## Risks

- An attacker can upload arbitrary file types, bypassing the intended restriction to
  "safe" document formats (PDF, ZIP). Depending on how uploaded files are stored and
  used, this could enable stored malicious content (e.g. scripts, executables) to reach
  other users or systems.
- Business impact: potential for malware distribution, storage abuse, or further attacks
  chained from an unrestricted upload (e.g. if uploaded files are later opened or
  executed by staff reviewing complaints).

## Actions

- Validate the file type server-side using both the file extension and its actual
  content/MIME type (not just what the client claims), rejecting anything outside the
  allowed set.
- Store uploaded files outside of any directly executable location, and serve them with
  headers that prevent them from being interpreted as scripts by browsers
  (e.g. `Content-Disposition: attachment`, `X-Content-Type-Options: nosniff`).
- Consider scanning uploaded files for malware before they are made available to staff or
  other users.