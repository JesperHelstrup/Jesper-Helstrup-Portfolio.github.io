---
tags:
  - Data
---
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

This gives us:

- a dedicated `ScanImages` table keyed by `ScanId`,
- a one-to-one relation,
- and cascade delete so removing a scan also removes its image.

Persisting an image now happens inside the same database transaction as the scan itself.

---

#### Service methods that attach images without forcing them into every query

`AddScanWithImageAsync` maps the incoming DTO to a `ScanImage`, attaches it to the `Scan`, and saves everything in a single `SaveChangesAsync` call.

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
~~~

When we want to load an image in the app, we only return one image, that way we never load more into the app than we absolutely need.

~~~
public async Task<ScanImageDto> GetScanImageAsync(
    Guid reportId,
    Guid scanId,
    CancellationToken ct = default)
{
	var report = await _reportRepo.GetWithScansAndImagesAsync(reportId, ct)
	?? throw new KeyNotFoundException($"Report with ID {reportId} not found.");
	
	var scan = report.Scans.FirstOrDefault(s => s.ScanId == scanId)
	?? throw new KeyNotFoundException($"Scan with ID {scanId} not found in report {reportId}.");
	
	var image = scan.ScanImage
	?? throw new KeyNotFoundException($"Scan with ID {scanId} does not have an image.");

return _mapper.Map<ScanImageDto>(image);
}
~~~

so listing scans is always fast, and downloading a single image is an intentional, separate call.

At the same time, we still have separate methods/endpoints for:

~~~
public interface IReportService : IBaseCrudService<ReportDto>

{
	Task<IReadOnlyList<ScanDto>> ListScansAsync(Guid reportId, CancellationToken ct = default);
	
	Task<Guid> AddScanAsync(Guid reportId, ScanDto dto, CancellationToken ct = default);
	
	Task UpdateScanAsync(Guid reportId, Guid scanId, ScanDto dto, CancellationToken ct = default);
	
	Task DeleteScanAsync(Guid reportId, Guid scanId, CancellationToken ct = default);
	
	Task<Guid> AddScanWithImageAsync(Guid reportId, ScanDto scanDto, ScanImageDto imageDto, CancellationToken ct = default);
	
	Task UpdateScanAndImageAsync(Guid reportId, Guid scanId, ScanDto scanDto, ScanImageDto imageDto, CancellationToken ct = default);
	
	Task<ScanImageDto> GetScanImageAsync(Guid reportId, Guid scanId, CancellationToken ct = default);

}
~~~

So we get the convenience of blob storage in the DB, but we’re not forced to load megabytes of image data every time the client just wants a list of scans.

### What I learned

This was mainly about making a practical trade-off rather than designing a “perfect” solution. Storing images directly in the database isn’t ideal at large scale, but for this project it simplified a lot of things:

- scans and images live in the same place,
- they’re saved in the same transaction, and
- backup/restore is straightforward.

The important part was realizing that the real performance problem isn’t “images in the database” by itself, but **pulling them when you don’t need them**. By keeping images in a separate table and exposing dedicated methods for loading them, we can fetch scan metadata quickly and only pay the cost of loading image bytes when a client actually asks for them.

If this system needed to scale further or serve a lot of image-heavy traffic, I’d probably move the binary data out to blob storage and keep just references in the database. But for now, this approach is good enough.