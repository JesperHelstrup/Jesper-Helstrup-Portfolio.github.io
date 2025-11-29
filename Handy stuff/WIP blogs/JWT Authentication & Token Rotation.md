### Purpose of the Implementation

For Authentication, I wanted to look into options that didn't involve defaulting to .NET's own Identity tool, since i have worked with that in a previous project. 

First i needed to decide what made the most sense for the project. The two main competitors i saw talked about in this context was server-side sessions and JWT tokens.

#### Server-Side Sessions
Server-side sessions are the classic approach — the server stores an active session for each logged-in user and the client just holds a session cookie.

**Pros:

- **Simple to understand** - Server keeps track of who’s logged in. Easy mental model.
- **Secure by default** - Cookies + server memory means no signed tokens to tamper with.
- **Good for traditional web apps** - When the backend and frontend live in the same domain, it works great.

**Cons:

- **Backend must store session state** - Every login adds memory usage.
- **Doesn’t fit APIs well** - Mobile apps and REST APIs don’t naturally use browser cookies.
- **Scaling becomes annoying** - If you add more backend instances, you need sticky sessions or distributed caching.
- **Harder to revoke / rotate gracefully** - Session invalidation is manual and messy.

#### JWT Tokens

JWT (JSON Web Token) pushes the “session” out to the client.  
The server just verifies signatures — no state stored.

**Pros:

- **Stateless** - Backend doesn’t track who’s logged in.
- **Perfect for mobile apps + APIs** - Tokens travel with requests, no cookies needed.
- **Easy to scale horizontally** - Any server instance can validate tokens without syncing session data.
- **Supports roles & claims** - You can embed useful info right inside the token.
- **Access + refresh token model is clean** - Short-lived access token + long-lived refresh token = modern, secure flow.

**Cons:

- **If you misconfigure or expose the signing key, it’s bad** - But that’s true for any auth system.
- **Tokens must expire** - Long-lived JWTs are dangerous, so you need refresh tokens / rotation.
- **A bit more setup compared to .NET Identity** - But still manageable.

I decided to go with JWT, mainly for the scaleability and the stateless design. It also seemed to be more suggested online and even though it required more setup, thats just extra learning for me.
### What I built

We ended up splitting the authentication logic across two services:

- `TokenService` → creates JWTs + refresh tokens
- `AuthService` → handles login + refresh flow
- `RefreshToken` model → keeps track of validity, revocation, and rotation history

The access token is our JWT.  
It’s short-lived and contains claims like:

- UserId
- Username
- Email
- Role

The backend signs the token with **HMAC-SHA256**, using a symmetric key from configuration:

~~~
var token = handler.CreateToken(new SecurityTokenDescriptor
{
    Issuer = _opt.Issuer,
    Audience = _opt.Audience,
    Subject = new ClaimsIdentity(claims),
    Expires = expires,
    SigningCredentials = new SigningCredentials(
        new SymmetricSecurityKey(_key), SecurityAlgorithms.HmacSha256)
});
~~~

The refresh token is different — it’s just a **random 256-bit value** encoded with Base64Url:

~~~
var bytes = RandomNumberGenerator.GetBytes(32);
var token = Base64UrlEncoder.Encode(bytes);

return new RefreshToken
{
    UserId = userId,
    Token = token,
    ExpiresUtc = DateTime.UtcNow.AddDays(days),
    CreatedByIp = ip
};
~~~

In summary:
- Access tokens → short-lived, contain identity
- Refresh tokens → long-lived, stored server-side, revocable

---

After hashing verification (see [[Hashing in Csharp With BCrypt]]), the login service issues:

- A fresh access token
- A fresh refresh token (saved in the database)

~~~
var (access, expUtc) = _tokens.CreateAccessToken(u);
var rt = _tokens.CreateRefreshToken(u.UserId, days: 14, ip: ip);

await _refreshTokenRepository.AddAsync(rt);
await _refreshTokenRepository.SaveChangesAsync();

return new TokenResponse(
    AccessToken: access,
    ExpiresInSeconds: (int)Math.Max(0, (expUtc - DateTime.UtcNow).TotalSeconds),
    RefreshToken: rt.Token
);

~~~

Storing refresh tokens means we can:
- Revoke them
- Track usage
- Detect abuse
- Rotate them later

This is basically the foundation for a secure token lifecycle.

---

Refresh tokens can’t live forever.  
If someone steals one, we want it to be **useless after a single use**.

That’s what rotation solves.

Here’s how the refresh logic works:

~~~
var existing = await _refreshTokenRepository.GetAsync(refreshToken);
if (existing is null || !existing.IsActive) return null;

var user = (await _repo.ListAsync()).FirstOrDefault(x => x.UserId == existing.UserId);
if (user is null) return null;

var replacement = _tokens.CreateRefreshToken(existing.UserId, days: 14, ip: ip);
existing.RevokedUtc = DateTime.UtcNow;
existing.ReplacedByToken = replacement.Token;
await _refreshTokenRepository.AddAsync(replacement);
await _refreshTokenRepository.SaveChangesAsync();

var (access, expUtc) = _tokens.CreateAccessToken(user);
return new TokenResponse(...);

~~~

What’s happening here?

- The old refresh token is immediately **revoked**
- A new refresh token replaces it
- Any stolen/old token becomes useless
- We build an audit trail:
    - When it was used
    - When it was revoked
    - Which new token replaced it

This is the recommended pattern from the OWASP guidelines.

---

The model includes everything we need to track validity:

~~~
public sealed class RefreshToken

{
	public Guid Id { get; set; } = Guid.NewGuid();
	
	public Guid UserId { get; set; }
	
	public string Token { get; set; } = default!;
	
	public DateTime ExpiresUtc { get; set; }
	
	public DateTime CreatedUtc { get; set; } = DateTime.UtcNow;
	
	public string? CreatedByIp { get; set; }
	
	public DateTime? RevokedUtc { get; set; }
	
	public string? ReplacedByToken { get; set; }
	
	public bool IsActive => RevokedUtc is null && DateTime.UtcNow < ExpiresUtc;

}
~~~

The `IsActive` helper keeps the AuthService clean:

- If token expired → `IsActive = false`
- If token was replaced → `IsActive = false`
- If token was revoked → `IsActive = false`

Basically:  
If the token isn’t valid in every possible way → throw it out.

---

JWTs are only useful if the backend **validates them** the same way it creates them.

Program.cs connects everything so the middleware uses the same key + issuer + audience:

~~~
builder.Services.AddScoped<ITokenService, TokenService>();
builder.Services.AddScoped<IAuthService, AuthService>();

builder.Services
    .AddAuthentication(JwtBearerDefaults.AuthenticationScheme)
    .AddJwtBearer(o => {
        o.TokenValidationParameters = new TokenValidationParameters {
            ValidateIssuer = true,
            ValidateAudience = true,
            ValidateLifetime = true,
            ValidateIssuerSigningKey = true,
            ValidIssuer = jwt.Issuer,
            ValidAudience = jwt.Audience,
            IssuerSigningKey = new SymmetricSecurityKey(Encoding.UTF8.GetBytes(jwt.Key)),
            ClockSkew = TimeSpan.FromMinutes(1)
        };
    });

~~~

Once this is configured, every `[Authorize]` endpoint now requires a valid access token.

### What I learned

This was definitely one of the heavier topics for me, mostly because there are so many little pieces that all rely on each other.  
But after working through it, I finally started to understand what’s actually going on behind the scenes when a modern app logs a user in.

The process essentially goes something like this:
- User logs in with its 


---
text for inspiration for "What i learned":

This was definitely one of the heavier topics for me, mostly because there are so many little pieces that all rely on each other.  
But after working through it, I finally started to understand what’s actually going on behind the scenes when a modern app logs a user in.

First thing I learned is that **a login system isn’t just “check password → done.”**  
There’s a whole little ecosystem involved:

- creating the JWT
    
- configuring issuer, audience, lifespan
    
- signing it correctly
    
- validating it in middleware
    
- issuing and storing refresh tokens
    
- revoking/rotating those tokens
    
- tracking which token replaced which
    
- returning everything to the client
    
- handling the refresh flow securely
    

Once I saw how all these parts fit together, JWTs suddenly made a lot more sense.

I also learned why **refresh tokens are even a thing**.  
Before this, I thought “why not just make the JWT last longer?” But that turns out to be terrible practice.  
Short-lived access tokens + long-lived refresh tokens is what actually keeps the system safe.  
And rotation was another thing I didn’t know existed — but now it makes complete sense.  
If someone steals a refresh token, rotation makes that stolen token useless the moment it’s used.

Another thing that clicked for me was how **claims** work.  
I knew JWTs contained data, but I didn’t fully get how that data gets into the token and how the downstream API reads it.  
After implementing it, the idea of embedding things like `UserId` and `Role` inside the token feels really natural.

Finally, setting up the ASP.NET validation pipeline taught me how picky JWT validation actually is.  
The middleware checks _everything_ — issuer, audience, the signing key, expiration — and rejects anything even slightly off.  
It gave me a good appreciation for how easy it is to accidentally misconfigure an authentication flow.

Even though I definitely had help building this part, I still feel like I came out of it with a much stronger understanding of how real-world authentication works — not just the “hello world” version.