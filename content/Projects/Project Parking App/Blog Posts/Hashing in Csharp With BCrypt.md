---
tags:
  - CyberSec
---
### Purpose of the Implementation

During research, I came across multiple sources explaining that **hashing passwords** is a fundamental security practice. The whole point is that the real password is **never stored anywhere**, not even by the backend developer or the database administrator.

If a database ever leaked—or if someone internal had access to it—hashing ensures they can’t just grab a user’s password and log in as them elsewhere.


#### Small Theory Section
Hashing is essentially a **one-way cryptographic transformation**:  
Turning something like `"password123"` into `$2a$12$HkLroV2eXFX8q7cA1tXD0uByP81EiPxQJ6mTz1VNVqFG5Of6nT3Vu` (perl hashing).

You can verify it, but you can’t reverse it.

There are many hashing algorithms (SHA-256, PBKDF2, scrypt), but for passwords, **BCrypt** was widely recommended.

**BCrypt** has three important properties:

- **Slow by design** → makes brute-force exponentially harder
- **Salted automatically** → same password never hashes to the same output
- **Adaptive work factor** → you can increase the cost as hardware improves

All three make BCrypt ideal for keeping passwords safe.

### What I built

I started by designing a passwordhasher class:

```
public sealed class BcryptPasswordHasher : IPasswordHasher
{
    public bool Verify(string password, string storedHash)
        => BCrypt.Net.BCrypt.Verify(password, storedHash);

    public string Hash(string password)
        => BCrypt.Net.BCrypt.HashPassword(password, workFactor: 12);

    public bool NeedsRehash(string storedHash)
        => BCrypt.Net.BCrypt.PasswordNeedsRehash(storedHash, newMinimumWorkLoad: 12);
}
```

This lets us easily generate password hashes and verify them later.  
The `workFactor: 12` setting controls how slow BCrypt is — higher = more secure.

---

Whenever a new user is created, we hash the password before saving:
~~~
public override async Task<Guid> CreateAsync(UserDto dto, CancellationToken ct = default)
{
    var user = _mapper.Map<User>(dto);
    if (!string.IsNullOrWhiteSpace(dto.Password))
    {
        user.Password = _hasher.Hash(dto.Password);
    }
    await _repo.AddAsync(user, ct);
    await _uow.SaveChangesAsync(ct);
}
~~~
(Same guard/assignment exists in UpdateAsync to handle password rotations.)

This ensures that **no plaintext password ever reaches the database.**

---

When a user tries to log in, we never compare the raw password.  
Instead, we compare the hash of the entered password with the stored hash:

~~~
public async Task<TokenResponse?> LoginAsync(string usernameOrEmail, string password, string? ip = null)

{
	var users = await _repo.ListAsync();
	
	var u = users.FirstOrDefault(x => x.Email == usernameOrEmail || x.Username == usernameOrEmail);
	
	if (u is null) return null;
	
	if (!_hasher.Verify(password, u.Password)) return null;
	
	var (access, expUtc) = _tokens.CreateAccessToken(u);
	
	var rt = _tokens.CreateRefreshToken(u.UserId, days: 14, ip: ip);
	
	await _refreshTokenRepository.AddAsync(rt);
	
	await _refreshTokenRepository.SaveChangesAsync();
	
	return new TokenResponse(
		AccessToken: access,
		ExpiresInSeconds: (int)Math.Max(0, (expUtc -DateTime.UtcNow).TotalSeconds),
		RefreshToken: rt.Token
	);

}
~~~

This way:

- The actual password is never stored
- The actual password is never logged
- The actual password is never transmitted back
- Only hashes are ever used server-side

This is considered industry-standard practice.

### What I learned

While digging into the topic, I initially struggled with one specific thing:  
**How can BCrypt tell if a password is correct when the same password produces _different_ hashes every time?**

If the hash changes every time, and we don’t store a separate salt, how on earth does the system know which salt to use later?

It's brilliantly simply, actually.
**BCrypt hides all the required information inside the final hash.**  
The salt isn’t lost — it’s embedded directly into the stored string.

A BCrypt hash looks like this:

`$2a$12$A9pKq4vrf2UP0V3F0yPjY.2eC8sZrDV1d6ZgWl5FX7xH0O7lBazGa`

And each piece has meaning:

- `$2a$` → BCrypt version
- `12$` → cost factor (how slow it is)
- `A9pKq4vrf2UP0V3F0yPjY.` → **this is the salt**
- `2eC8sZrDV1d6ZgWl5FX7xH0O7lBazGa` → the actual hash result    

When `BCrypt.Verify()` runs:

1. It reads the version, cost, and **salt** straight from the stored hash
2. Hashes the incoming password with those variables
3. Compares the results

As a conclusion, because of how easy it was to implement compared to how much it helps if something like SQL injection ever exposes user rows — or if a backup leaks, logs get misconfigured, or anything else goes sideways — BCrypt makes the stolen data mostly useless and unreadable. So honestly, it feels like a no-brainer, and I’m happy I got to try it out.
