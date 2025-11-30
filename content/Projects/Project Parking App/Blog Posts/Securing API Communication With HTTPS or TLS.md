---
tags:
  - CyberSec
---
### Purpose of the Implementation

The app sends **sensitive data** from the phone to the backend — license plates, GPS coordinates, timestamps, user identity, and sometimes images. Even if everything in the database is protected and the auth flow is solid, it doesn’t help much if someone can just sit on the network and read the traffic in plain text.

So the purpose here is simple:

- Make sure all communication between client and backend goes over **HTTPS/TLS**.
- Prepare the backend so that, once certificates and hosting are in place, it can **reject or redirect plain HTTP** and only accept encrypted traffic.

This isn’t fully enabled yet in the prototype, but the pipeline is set up with that in mind.

### What I built

#### CORS policy that assumes an HTTPS frontend

~~~
builder.Services.AddCors(options =>
{
    options.AddPolicy("WebAppCors", policy =>
    {
        policy.WithOrigins("https://localhost:5173")
            .AllowAnyHeader()
            .AllowAnyMethod()
            .AllowCredentials();
    });
});
~~~

The CORS configuration explicitly allows an **HTTPS origin** for the web frontend. That means the backend is already expecting to talk to a secure frontend, not `http://localhost`.

When this runs in a real environment, the allowlist can be extended with the actual production HTTPS domains, but the intent is clear: the API is meant to be called from secure origins only.

---
#### HTTPS redirection middleware wired in, but not yet enabled

~~~
// app.UseHttpsRedirection();
~~~

ASP.NET’s HTTPS redirection middleware is already in `Program.cs`, just commented out.

Once the app is deployed behind a valid certificate (e.g. reverse proxy, cloud hosting), turning this on will:

- redirect all incoming `http://` requests to `https://`, and
- make it much harder for clients to “accidentally” talk to the API over plain HTTP.

So even though the current dev setup still runs without enforced HTTPS, the structure is ready for a production setup where **HTTP is only a redirect to HTTPS**.
### What I learned

CORS was already something we covered earlier in the education, so there wasn’t much new on that part. What _was_ useful was how this topic connected directly to the work I did in [[Symmetric vs Asymmetric Encryption]]. While looking into HTTPS, I realized it relies on asymmetric cryptography (like RSA) during the TLS handshake to negotiate a symmetric session key.

Aside from that, this was mostly just a small reminder that once real certificates and hosting are in place, enabling HTTPS in ASP.NET is straightforward.