### **Why BCrypt Is Designed for Password Security**

BCrypt isn’t just “a way to hash things.”  
It’s specifically designed for **protecting passwords**, and it does this using three key properties:

---

#### **1️⃣ Slow by Design (Computational Cost)**

Traditional hashing algorithms like SHA-256 are extremely fast.  
Fast is good for checksums — but terrible for password security.

Because if a hacker steals your hash database, they can try billions of guesses per second using GPUs or ASICs.  
With SHA-256, they can brute-force enormous amounts of passwords extremely quickly.

BCrypt is deliberately **slow**, meaning:

- Each password guess takes significantly longer
- Attackers can’t attempt billions of guesses per second
- Even with powerful hardware, brute-forcing becomes expensive and time-consuming

This slowing effect is the core of BCrypt’s protection.

---

#### **2️⃣ Automatic Salting (Randomized Hashes)**

BCrypt automatically generates a **unique random salt** for each password.

A salt is random data added to the input before hashing.

This prevents:

- **Rainbow table attacks**
- **Precomputed hash lookups**
- Matching passwords across users
- Two identical passwords producing identical hashes

Example:

|Password|Hash|
|---|---|
|`Password123`|`$2a$12$Kq1Xn…`|
|`Password123`|`$2a$12$d83lO…`|

Same password → **completely different hash**, every time.

This means attackers can’t take shortcuts or reuse precomputed databases of hashes.

---

#### **3️⃣ Adaptive Work Factor (Configurable Cost)**

BCrypt includes a tunable **work factor** (sometimes called cost).

In your code:

`BCrypt.Net.BCrypt.HashPassword(password, workFactor: 12);`

That `12` determines how slow the hash computation is.

As computers get faster over the years, developers can simply **increase the work factor**, and the hashing becomes slower accordingly.

This means:

- BCrypt gets stronger as hardware improves
    
- You can adjust security without changing algorithms
    
- Old hashes can be detected and rehashed using `NeedsRehash`
    

This future-proofs your system.

---

### **Why These Three Properties Matter Together**

If we put it all together:

- **Slow** → brute-forcing becomes impractical
    
- **Salted** → precomputed attacks are useless
    
- **Adaptive** → protection stays strong as CPUs improve
    

This combination is exactly why BCrypt is recommended by:

- OWASP
    
- NIST
    
- Most modern security guidelines
    
- Practically all reputable authentication frameworks
    

And why it’s the right choice for your project.

# **How BCrypt Keeps Track of the Salt**

When BCrypt hashes a password, it produces one single string that contains:

1. **The algorithm version**
    
2. **The work factor (cost)**
    
3. **The salt**
    
4. **The final hash**
    

All in one long encoded string.

A BCrypt hash format looks like this:

`$2a$12$A9pKq4vrf2UP0V3F0yPjY.2eC8sZrDV1d6ZgWl5FX7xH0O7lBazGa`

Breaking it down:

- `$2a$` → BCrypt version
    
- `12$` → cost factor
    
- `A9pKq4vrf2UP0V3F0yPjY.` → **this is the salt**
    
- `2eC8sZrDV1d6ZgWl5FX7xH0O7lBazGa` → final password hash
    

So the salt isn’t lost.  
It’s literally _embedded_ inside the stored hash.