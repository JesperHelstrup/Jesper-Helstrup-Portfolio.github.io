### Week 38

- Learn: Encryption basics (symmetric vs asymmetric).
- Research: What’s the role of encryption in protecting database backups?
#### Tasks

- [ ] Demo: write a small C# console program that encrypts and decrypts a text file with `Aes` and show the before/after.
- [ ]  Demo: write C# code to generate an RSA key pair, sign a “parking report” string, and verify the signature.
- [ ]  Short note: symmetric vs asymmetric encryption – when to use each.
- [ ]  Mini reflection: risks of leaving Postgres backups unencrypted.

### Week 39
- Learn: Authentication vs Authorization (RBAC, JWT, OAuth2).
- Hands-on: implement JWT authentication in an ASP.NET Core minimal API.
- Research: Common DB attack vectors (SQL injection, privilege escalation).
#### Tasks
- [ ] API demo: screenshot showing 401 without JWT, 200 with valid JWT.
- [ ] Short note: how SQL injection works and how parameterized queries block it.
- [ ] Write-up: compare RBAC vs JWT/OAuth2 in one paragraph each.

### Week 40
- Learn: Hashing vs encryption (SHA-256, bcrypt/argon2).
- Hands-on: hash passwords in C# with bcrypt/argon2.
- Research: Database attack types (SQL injection, privilege escalation, brute force).
#### Tasks
- [ ] Code snippet showing a password hash + verification in C#.
- [ ] Short note: differences between hashing, salting, and encryption.
- [ ] Reflection: “What could happen if passwords were stored unhashed in my app?”

### Week 41
- Learn: TLS basics (HTTPS, certificates, handshake).
- Hands-on: secure a test ASP.NET Core minimal API with HTTPS.
- Learn: Digital signatures (concept + use in signed reports/PDFs).
- Hands-on: generate a signed message with OpenSSL.
#### Tasks
- [ ] Demo: API call failing without HTTPS, succeeding with HTTPS.
- [ ] Signed message screenshot (OpenSSL) + explanation of digital signatures.
- [ ] Reflection: why signed reports matter in legal/forensic contexts (fits your app).