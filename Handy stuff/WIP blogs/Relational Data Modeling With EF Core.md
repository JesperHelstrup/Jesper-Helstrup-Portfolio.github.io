---
tags:
  - Data
---
### Purpose of the Implementation

The goal of the data model in this project was to represent the structure of a roadwork job, the scans made on-site, the optional images, and the surrounding organization (teams, departments, recipients).

This meant designing a relational schema that:

- matches the domain (reports → scans → images),
- enforces parent/child relationships,
- keeps data consistent (especially when deleting),
- works cleanly with EF Core’s tracking and transactions,
- and can be queried efficiently from the app.

Most of this was familiar territory since I’ve worked with EF Core before, but the project still required a clean and well-structured model that supports everything else the app is doing.

### What I built

#### A parent entity that holds a collection of children

~~~
public class Report
{
    public Guid ReportId { get; set; }
    ...
    public ICollection<Scan> Scans { get; set; } = new List<Scan>();
}
~~~

A `Report` represents a roadwork job. It holds a collection of `Scans`, which models the one-to-many relationship:  
**one report → many scans**.

This is the basis for grouping all the detected cars under the correct job.

---
#### An entity that is both a child and a parent (typical in deeper models)

~~~
public class Team
{
    public Guid TeamId { get; set; }
    public required string Name { get; set; }

    public Guid DepartmentId { get; set; }
    public Department Department { get; set; } = null!;

    public ICollection<User> Users { get; set; } = new List<User>();
}
~~~

`Team` sits in the middle of the hierarchy:

- It has a **foreign key** to `Department` (child side).
- It has a **collection of Users** (parent side).

This shows how EF Core handles relational depth — one table can be both a reference and a parent without any special configuration.

---
#### Fluent configuration for relationships and delete behaviors

~~~
b.Entity<ScanImage>()
    .HasKey(si => si.ScanId);

b.Entity<Scan>()
    .HasOne(s => s.ScanImage)
    .WithOne(si => si.Scan)
    .HasForeignKey<ScanImage>(si => si.ScanId)
    .OnDelete(DeleteBehavior.Cascade);

b.Entity<Scan>()
    .HasOne<Report>()
    .WithMany(r => r.Scans)
    .HasForeignKey(s => s.ReportId)
    .OnDelete(DeleteBehavior.Cascade);
~~~

Here we define:

- `ScanImage` uses `ScanId` as both its **PK** and **FK** → a _true_ one-to-one.
- A scan’s image will be deleted along with the scan (`Cascade`).
- Reports cascade-delete their scans as well.

This keeps the database clean and prevents orphaned records if a report or scan gets removed.

---
#### Defining the tables through DbSet<> properties

~~~
public DbSet<Admin> Admins { get; set; }
public DbSet<Department> Departments { get; set; }
public DbSet<Recipient> Recipients { get; set; }
public DbSet<Report> Reports { get; set; }
public DbSet<Scan> Scans { get; set; }
public DbSet<ScanImage> ScanImages { get; set; }
public DbSet<Team> Teams { get; set; }
public DbSet<User> Users { get; set; }
public DbSet<RefreshToken> RefreshTokens { get; set; }
~~~

These properties tell EF Core which aggregates to track and map to tables. They also form the entry point for all LINQ queries.

This is the final layer that ties the C# model to the relational database.

### What I learned

Because I’ve worked with EF Core before, most of this wasn’t new. It was mainly about shaping the data model around what the app actually needed:

- grouping scans under reports,
- supporting optional images,
- keeping references clean,
- choosing cascade behavior that makes sense, and
- making sure the model is easy to query from the services.

It was also a good reminder that EF Core’s conventions cover a lot, but fluent configuration is still important when you want precise control — especially for one-to-one relationships and delete rules.