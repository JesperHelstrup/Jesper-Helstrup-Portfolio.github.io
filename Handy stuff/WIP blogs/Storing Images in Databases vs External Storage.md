#Data 
### Purpose of the Implementation

For this project we needed a way to attach photos to scans so they could be included as part of the documentation. The main question was **where** to store those images:

- directly in the database as binary data, or
- externally (disk, blob storage, device-only), with only references in the DB.

For now, we chose to store the image bytes in Postgres. The goal wasn’t to design the perfect long-term storage architecture, but to:

- keep scans and their images **inside the same transaction**,
- keep the implementation simple enough for the project scope, and
- still avoid slowing everything down when listing scans by **not** always loading the image with the metadata.

### What I built

#### A one-to-one Scan ↔ ScanImage model in the database

~~~
public class Scan
{
    public Guid ScanId { get; set; }
    // ...
    public Guid ReportId { get; set; }

    public ScanImage? ScanImage { get; set; }
}

public class ScanImage
{
    public Guid ScanId { get; set; }

    public required byte[] ImageData { get; set; }
    public required string ContentType { get; set; }
    public DateTime CreatedAt { get; set; }

    public Scan? Scan { get; set; }
}
~~~

Each `Scan` can have an associated `ScanImage`, which stores:

- the raw image bytes (`byte[] ImageData`),
- the MIME type (`ContentType`), and
- a timestamp (`CreatedAt`).

So instead of putting a big blob column directly on the `Scans` table, images live in a separate table but are still tied 1:1 to a scan.

---

#### EF Core configuration and migration for the image table

~~~
public DbSet<ScanImage> ScanImages { get; set; }

// ...

b.Entity<ScanImage>().HasKey(si => si.ScanId);

b.Entity<Scan>()
    .HasOne(s => s.ScanImage)
    .WithOne(si => si.Scan)
    .HasForeignKey<ScanImage>(si => si.ScanId)
    .OnDelete(DeleteBehavior.Cascade);

~~~

Migration:
~~~
migrationBuilder.CreateTable(
    name: "ScanImages",
    columns: table => new
    {
        ScanId = table.Column<Guid>(type: "uuid", nullable: false),
        ImageData = table.Column<byte[]>(type: "bytea", nullable: false),
        ContentType = table.Column<string>(type: "text", nullable: false),
        CreatedAt = table.Column<DateTime>(type: "timestamp with time zone", nullable: false)
    },
    constraints: table =>
    {
        table.PrimaryKey("PK_ScanImages", x => x.ScanId);
        table.ForeignKey(
            name: "FK_ScanImages_Scans_ScanId",
            column: x => x.ScanId,
            principalTable: "Scans",
            principalColumn: "ScanId",
            onDelete: ReferentialAction.Cascade);
    });
~~~

This gives us:

- a dedicated `ScanImages` table keyed by `ScanId`,
- a one-to-one relation,
- and cascade delete so removing a scan also removes its image.

Persisting an image now happens inside the same database transaction as the scan itself.

---

#### Service methods that attach images without forcing them into every query

~~~
public async Task<Guid> AddScanWithImageAsync(
    Guid reportId,
    ScanDto scanDto,
    ScanImageDto imageDto,
    CancellationToken ct = default)
{
    // ... map scan and attach to report

    var scanImage = _mapper.Map<ScanImage>(imageDto);
    scanImage.ScanId = scan.ScanId;
    scan.ScanImage = scanImage;

    report.Scans.Add(scan);
    await _uow.SaveChangesAsync(ct);

    return scan.ScanId;
}

public async Task<ScanImageDto> GetScanImageAsync(
    Guid reportId,
    Guid scanId,
    CancellationToken ct = default)
{
    // ... fetch only the image for that scan
}
~~~

`AddScanWithImageAsync` maps the incoming DTO to a `ScanImage`, attaches it to the `Scan`, and saves everything in a single `SaveChangesAsync` call.

At the same time, we still have separate methods/endpoints for:

- listing scans **without** images (fast, small payloads), and
- fetching **only the image** when needed.

So we get the convenience of blob storage in the DB, but we’re not forced to load megabytes of image data every time the client just wants a list of scans.

### What I learned

This was mainly about making a practical trade-off rather than designing a “perfect” solution. Storing images directly in the database isn’t ideal at large scale, but for this project it simplified a lot of things:

- scans and images live in the same place,
- they’re saved in the same transaction, and
- backup/restore is straightforward.

The important part was realizing that the real performance problem isn’t “images in the database” by itself, but **pulling them when you don’t need them**. By keeping images in a separate table and exposing dedicated methods for loading them, we can fetch scan metadata quickly and only pay the cost of loading image bytes when a client actually asks for them.

If this system needed to scale further or serve a lot of image-heavy traffic, I’d probably move the binary data out to blob storage and keep just references in the database. But for now, this approach is good enough.