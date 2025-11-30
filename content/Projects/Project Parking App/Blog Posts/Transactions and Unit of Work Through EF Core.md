---
tags:
  - Data
---
### Purpose of the Implementation

Even though EF Core already handles transactions internally, I still wanted a clean way to coordinate database changes across different repositories. The goal wasn’t to reinvent EF’s transaction system, but to give the application a single point where changes are committed. That keeps the code predictable and avoids situations where half-finished writes slip into the database.

Using a Unit of Work also makes the service layer more explicit:  
**“Nothing is saved until SaveChanges is called.”**

### What I built

#### A simple Unit of Work wrapper around AppDbContext

~~~
public class UnitOfWork : IUnitOfWork
{
    private readonly AppDbContext _db;

    public UnitOfWork(AppDbContext db)
    {
        _db = db;
    }

    public Task<int> SaveChangesAsync(CancellationToken ct = default)
    {
        return _db.SaveChangesAsync(ct);
    }
}
~~~

The `UnitOfWork` is intentionally minimal. It doesn’t create transactions manually — it simply exposes `SaveChangesAsync`, which triggers EF Core’s built-in transaction behavior.

Every service receives the same scoped `AppDbContext`, so calling `SaveChangesAsync` commits _all_ pending changes for that request in a single transaction.

---

#### A CRUD service layer that stages changes, then commits them through the Unit of Work

~~~
public virtual async Task<Guid> CreateAsync(TDto dto, CancellationToken ct = default)
{
    var entity = _mapper.Map<TEntity>(dto);
    await _repo.AddAsync(entity, ct);
    await _uow.SaveChangesAsync(ct);
    return GetEntityId(entity);
}

public virtual async Task UpdateAsync(TDto dto, CancellationToken ct = default)
{
    var entity = await _repo.GetByIdAsync(GetDtoId(dto), ct)
        ?? throw new KeyNotFoundException(...);

    _mapper.Map(dto, entity);
    await _repo.UpdateAsync(entity);
    await _uow.SaveChangesAsync(ct);
}
~~~

All CRUD operations follow the same pattern:

1. Retrieve or map the entity
2. Stage the changes in the repository (but nothing hits the DB yet)
3. Call `_uow.SaveChangesAsync` to commit

If anything inside `SaveChangesAsync` fails, EF rolls back the whole transaction, and the controller never sees a half-applied update.

---

#### Domain logic follows the same transactional pattern

~~~
var report = await _reportRepo.GetWithScansAsync(reportId, ct)
    ?? throw new KeyNotFoundException(...);

report.Scans.Add(scan);

await _uow.SaveChangesAsync(ct);

~~~

In the `ReportService`, adding a scan updates the tracked object graph by attaching a new `Scan` to an existing `Report`. EF Core marks only the `Scan` entity as new, and when `SaveChangesAsync` is called it generates the INSERT for that scan in a single transaction.

It’s the same pattern as the CRUD service: you change tracked entities first, then commit everything in one go through the Unit of Work.

### What I learned

Even though EF Core handles the actual transaction mechanics for you, having a Unit of Work layer makes things more explicit and predictable. I didn’t learn anything fundamentally new here, but it reinforced the value of having one clear commit point in the application instead of letting different services save changes independently.

It also reminded me how EF Core batches INSERT/UPDATE/DELETE operations together automatically. As long as everything happens through the same DbContext and SaveChanges is called once, you get atomic behavior without writing transaction code manually.