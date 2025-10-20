### 1) HTTPS only (no cleartext)

- **Do:** Enforce HTTPS + HSTS on the ASP.NET Core API; block HTTP in Android network-security-config.
- **Base .NET?** Yes (ASP.NET Core built-in).
- **Effort:** **S (½ day)**.

### 2) Login + JWT for the API

- **Do:** Issue JWT on login; protect endpoints with `[Authorize]`; store token in app’s secure storage; attach `Authorization: Bearer`.
- **Base .NET?** Yes (`Microsoft.AspNetCore.Authentication.JwtBearer`, `System.IdentityModel.Tokens.Jwt`).
- **Effort:** **M (1–2 days)**.

### 3) Encrypt sensitive data at rest (photos + GPS)

- **Do:** Encrypt bytes before saving to local files/SQLite using **AES-GCM**; keep the key in platform keystore (Android) and wrap/unwrap it for use.
- https://www.postgresql.org/docs/current/encryption-options.html
- **Base .NET?** Crypto: **Yes** (`System.Security.Cryptography.AesGcm`).  
  Android keystore access: needs a tiny platform-specific call (MAUI/Xamarin binding or Java interop). Still “no extra libs”.
- **Effort:** **M (2–3 days)**.

### 4) Backups & storage hygiene

- **Do:** Private app storage only; exclude sensitive files from Android auto-backup; never write to shared storage.
- **Base .NET?** Yes (code), plus Android manifest flag.
- **Effort:** **S (½ day)**.

### 5) Minimal permissions (location)

- **Do:** Foreground location only; explain why in UI; no background unless truly required.
- **Base .NET?** Yes (app code + manifest).
- **Effort:** **S (½ day)**.

### 6) Logging hygiene

- **Do:** Centralize logging; never log tokens/plates/GPS; add a redaction helper.
- **Base .NET?** Yes (`ILogger`, filters).
- **Effort:** **S (½ day)**.

### 7) Audit trail (who scanned what/when)

- **Do:** DB table that records userId, time, deviceId, hash of image, GPS; write via EF Core.
- **Base .NET?** Yes (EF Core).
- **Effort:** **S–M (1 day)**.

### 8) Integrity on reports (hash or signature)

- **Good-enough now:** Compute **SHA-256** of each exported report payload; store hash in DB and print it on the PDF footer (“Integrity ID: …”).
- **“True PDF signature”?** Needs a PDF lib (3rd-party). With **base .NET only**, you can sign the **data** (CMS/PKCS#7) and ship a detached `.sig` file along the PDF.
- **Base .NET?** Hash/sign data: **Yes** (`SHA256`, `SignedCms`). “Sign the PDF file itself”: **No** without a PDF lib.
- **Effort:** Hash **S (½ day)**; detached signature **M (1–2 days)**; embedded PDF signature **L (3–5 days, needs lib)**.

### 9) (Optional) Certificate pinning

- **Do:** Pin your API cert/public key (Android network security config or custom `HttpClientHandler` pinning).
- **Base .NET?** Yes (handler), plus Android XML.
- **Effort:** **M (1 day)**.

### Data Storage
## A simple order to implement

1. **Identity roles** + protect endpoints.
2. **Least-privilege DB login**.
3. **RLS** with `app.user_id` set per request.
4. **Plate hash + encryption** (pgcrypto or app-side).
5. **Report hash**.
6. **Storage hygiene + retention**.