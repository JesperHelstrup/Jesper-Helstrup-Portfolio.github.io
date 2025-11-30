---
tags:
  - CyberSec
  - Data
---
### Purpose of the Implementation

SQL Injection is one of those classic vulnerabilities that never really dies. Any time you build something that talks to a database, you have to assume that user input is hostile until proven otherwise.

For this project, I didn’t want **controllers** or **services** building SQL strings manually. Instead, I wanted:

- All database access to go through **EF Core** using **LINQ**
- A clean **repository + service** layer between controllers and the database
- Input constraints at the API edge (routing & model binding)

The goal wasn’t to “trust EF Core blindly”, but to **use it in the way it’s designed to prevent SQL injection**: parameterized queries, no string-concatenated SQL, and strongly-typed queries end-to-end.

Disclaimer: Whenever i refer to HTTP in this blog post, it's because the current version allows for HTTP, the final product would have been locked to only using HTTPS. 
### What I built
#### Registering EF Core and repositories in Program.cs

~~~
builder.Services.AddDbContext<AppDbContext>(options =>
    options.UseNpgsql(builder.Configuration.GetConnectionString("AppDb"))
);

builder.Services.AddScoped(typeof(IBaseRepository<>), typeof(BaseRepository<>));
builder.Services.AddScoped<IReportRepository, ReportRepository>();
// ...
builder.Services.AddScoped<IAuthService, AuthService>();

~~~

By registering `AppDbContext` like this, each HTTP request gets its own **scoped** DbContext. All repositories receive that same context, so every query is built through EF Core.

Because everything goes through LINQ, **EF composes parameterized SQL under the hood**. The controllers never see SQL; they just work with services and repositories.

---

#### A generic repository that only uses EF Core APIs

~~~
public Task<T?> GetByIdAsync(Guid id, CancellationToken ct = default) =>
    _set.FindAsync(new object[] { id }, ct).AsTask();

public async Task<IReadOnlyList<T>> ListAsync(CancellationToken ct = default) =>
    await _set.AsNoTracking().ToListAsync(ct);

~~~

The `BaseRepository<T>` wraps common operations and **never concatenates SQL**. It passes values like `Guid id` into EF APIs such as `FindAsync` and `ToListAsync`.

Those methods generate queries where the input values are sent as **parameters**, not string fragments. That parameterization is the core defence against SQL injection, and any service using this repository automatically benefits.

---

#### Querying related data safely in ReportRepository

~~~
public Task<Report?> GetWithScansAsync(Guid id, CancellationToken ct = default)
{
    return _db.Reports
        .Include(r => r.Scans)
        .FirstOrDefaultAsync(r => r.ReportId == id, ct);
}

~~~

Here the query still stays as **pure LINQ**:

- `Include(r => r.Scans)` expresses eager loading
- `FirstOrDefaultAsync(r => r.ReportId == id)` expresses the filter

EF Core translates this into SQL and treats the `id` value as a parameter. Even though this query pulls in related entities (`Scans`), there is never a point where user input is concatenated into a raw SQL string.

---

#### Safe lookups for refresh tokens

~~~
public Task<RefreshToken?> GetAsync(string token) =>
    _db.Set<RefreshToken>().FirstOrDefaultAsync(x => x.Token == token);

public Task AddAsync(RefreshToken token)
{
    _db.Set<RefreshToken>().Add(token);
    return Task.CompletedTask;
}
~~~

Refresh tokens originate from HTTP requests and are therefore **untrusted input**.

But they only ever reach the database through `FirstOrDefaultAsync` with a predicate expression. EF Core turns `x.Token == token` into something like `WHERE Token = @p0` under the hood.

Again, no string concatenation, no chance for `"'; DROP TABLE RefreshTokens; --"` to become valid SQL.

---

#### Input constraints and validation at the controller level

~~~
[HttpGet("{id:guid}")]
public virtual async Task<IActionResult> GetById(Guid id, CancellationToken ct)
{
    var dto = await _service.GetByIdAsync(id, ct);
    return dto is null ? NotFound() : Ok(dto);
}

[HttpPut("{id:guid}")]
public virtual async Task<IActionResult> Update(Guid id, [FromBody] TDto dto, CancellationToken ct)
{
    if (id != GetDtoId(dto)) return BadRequest("Id mismatch");
    // ...
}
~~~

At the API edge:

- Route parameters are constrained to `Guid` via `{id:guid}`.
- ASP.NET Core model binding ensures `id` is already a `Guid` before it reaches the service.
- The PUT action validates that the route ID matches the DTO’s ID.

Together with EF Core’s parameterized queries, this means there’s **no direct path for arbitrary strings** to be turned into SQL fragments anywhere in the normal request flow.

---

Overall, the pattern is:

- Register a single `DbContext` per request
- Funnel all data access through **LINQ-based repositories**
- Use navigation properties and `Include` instead of custom SQL
- Validate and constrain input at controller level

A natural extension is that seeding and migrations (`DbSeeder`) also use the same `AppDbContext`, so even administrative code benefits from the same protections.
### What I learned

Because I’ve worked with EF Core in several previous projects, this part of the system didn’t introduce anything fundamentally new for me. It was mostly a chance to apply what I already knew in a more security-focused context.

The main thing this reinforced is that SQL injection prevention is really just about avoiding string-built SQL. As long as you stick to LINQ and let EF Core handle parameter binding, you automatically get stronger protection.

If I was using another backend framework without the same guarantees, I would have had to put more time into designing the equivalent protection myself or found another tool. But in .NET, EF Core already gives you what you need — you just have to use it correctly.

Overall, this part was mostly practice and consolidation rather than new learning, but still important given how common SQL injection attacks are.