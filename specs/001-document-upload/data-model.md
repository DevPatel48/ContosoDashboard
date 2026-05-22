# Data Model: Document Upload and Management

## Entities

### Document

Represents an uploaded file with metadata stored in the database and the physical file stored on the local filesystem.

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| DocumentId | int | Yes | Primary key, auto-increment integer (stakeholder requirement - not GUID) |
| Title | string (255) | Yes | User-provided document title |
| Description | string (1000) | No | Optional document description |
| Category | string (50) | Yes | Text category (not enum): Project Documents, Team Resources, Personal Files, Reports, Presentations, Other |
| FilePath | string (500) | Yes | GUID-based relative path to stored file (e.g., `documents/a1b2c3d4-e5f6-7890.pdf`) |
| OriginalFileName | string (255) | Yes | Original filename provided by user |
| FileSize | long | Yes | File size in bytes |
| FileType | string (255) | Yes | MIME type of the file |
| UploadedBy | string (450) | Yes | Navigation property to User (foreign key to User.Id) |
| UploadedDate | DateTime | Yes | Automatic timestamp of upload |
| AssociatedProjectId | int? | No | Nullable foreign key to Project.Id |
| Tags | string (500) | No | Comma-separated tags for searchability |
| IsDeleted | bool | Yes | Soft delete flag (default: false) - for future soft delete feature |

**Validation Rules**:
- Title: Required, max 255 characters, trimmed
- Category: Required, must be one of predefined values
- FileSize: Must be <= 25 MB (26,214,400 bytes)
- FileType: Must be in allowed MIME types list
- FilePath: Must be within storage root directory (path traversal check)

**Indexes**:
- `IX_Document_UploadedBy`: Index on UploadedBy for "My Documents" queries
- `IX_Document_Category`: Index on Category for filtering
- `IX_Document_AssociatedProjectId`: Index on AssociatedProjectId for project documents
- `IX_Document_Title`: Index on Title for search
- `IX_Document_UploadedDate`: Index on UploadedDate for sorting

**Relationships**:
- `Document` → `User` (many-to-one): UploadedBy
- `Document` → `Project` (many-to-one, optional): AssociatedProjectId
- `Document` → `DocumentShare` (one-to-many): Shared with users
- `Document` → `DocumentAuditLog` (one-to-many): Audit trail

---

### DocumentShare

Tracks sharing relationships between documents and users, enabling "Shared with Me" functionality.

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| DocumentShareId | int | Yes | Primary key, auto-increment integer |
| DocumentId | int | Yes | Foreign key to Document.DocumentId |
| UserId | string (450) | Yes | Foreign key to User.Id (the user being shared with) |
| SharedBy | string (450) | Yes | Foreign key to User.Id (the user who initiated sharing) |
| SharedDate | DateTime | Yes | Timestamp of share action |
| AccessLevel | string (20) | Yes | "View" or "Edit" (default: "View") |

**Validation Rules**:
- Cannot share a document with the document owner (already has access)
- Cannot create duplicate shares (same document + same user)
- AccessLevel: Must be "View" or "Edit"

**Indexes**:
- `IX_DocumentShare_UserId`: Index on UserId for "Shared with Me" queries
- `IX_DocumentShare_DocumentId`: Index on DocumentId for document's share list
- `UQ_DocumentShare_Document_User`: Unique index on (DocumentId, UserId) to prevent duplicates

**Relationships**:
- `DocumentShare` → `Document` (many-to-one): DocumentId
- `DocumentShare` → `User` (many-to-one): UserId (shared with)
- `DocumentShare` → `User` (many-to-one): SharedBy (shared by)

---

### DocumentAuditLog

Append-only audit trail for all document activities, supporting compliance and security requirements.

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| AuditLogId | int | Yes | Primary key, auto-increment integer |
| DocumentId | int | Yes | Foreign key to Document.DocumentId |
| UserId | string (450) | Yes | Foreign key to User.Id (user who performed the action) |
| ActionType | string (20) | Yes | "Upload", "Download", "Share", "Unshare", "Delete", "Edit", "Replace" |
| Timestamp | DateTime | Yes | Automatic timestamp of action |
| Details | string (1000) | No | Optional details (e.g., "Shared with user X", "Title changed from A to B") |

**Validation Rules**:
- ActionType: Must be one of predefined values
- Timestamp: Auto-set to DateTime.UtcNow on creation
- Details: Optional, max 1000 characters

**Indexes**:
- `IX_DocumentAuditLog_DocumentId`: Index on DocumentId for document audit history
- `IX_DocumentAuditLog_UserId`: Index on UserId for user activity history
- `IX_DocumentAuditLog_Timestamp`: Index on Timestamp for time-range queries

**Relationships**:
- `DocumentAuditLog` → `Document` (many-to-one): DocumentId
- `DocumentAuditLog` → `User` (many-to-one): UserId

---

## Entity Relationships Diagram

```
User (1) ────< (M) Document (1) ────> (M) DocumentShare
  |                         |
  |                    (M) DocumentAuditLog
  |
  └───> (M) DocumentShare (as "SharedBy")
  └───> (M) DocumentShare (as "UserId")
  └───> (M) DocumentAuditLog

Project (1) ────< (M) Document (via AssociatedProjectId)
```

## Database Migration Plan

### Migration 001_AddDocumentEntities.cs

**Operations**:
1. Create `Documents` table with all columns and indexes
2. Create `DocumentShares` table with all columns and indexes
3. Create `DocumentAuditLogs` table with all columns and indexes
4. Add foreign key constraints:
   - `Documents.UploadedBy` → `AspNetUsers.Id`
   - `Documents.AssociatedProjectId` → `Projects.Id` (nullable)
   - `DocumentShares.DocumentId` → `Documents.DocumentId` (cascade delete)
   - `DocumentShares.UserId` → `AspNetUsers.Id`
   - `DocumentShares.SharedBy` → `AspNetUsers.Id`
   - `DocumentAuditLogs.DocumentId` → `Documents.DocumentId` (cascade delete)
   - `DocumentAuditLogs.UserId` → `AspNetUsers.Id`

### ApplicationDbContext Extensions

```csharp
public DbSet<Document> Documents { get; set; }
public DbSet<DocumentShare> DocumentShares { get; set; }
public DbSet<DocumentAuditLog> DocumentAuditLogs { get; set; }

protected override void OnModelCreating(ModelBuilder modelBuilder)
{
    // Document configuration
    modelBuilder.Entity<Document>(entity =>
    {
        entity.HasKey(e => e.DocumentId);
        entity.Property(e => e.Title).IsRequired().HasMaxLength(255);
        entity.Property(e => e.Category).IsRequired().HasMaxLength(50);
        entity.Property(e => e.FilePath).IsRequired().HasMaxLength(500);
        entity.Property(e => e.FileType).IsRequired().HasMaxLength(255);
        entity.HasOne(e => e.Uploader).WithMany().HasForeignKey(e => e.UploadedBy);
        entity.HasOne(e => e.AssociatedProject).WithMany().HasForeignKey(e => e.AssociatedProjectId);
    });

    // DocumentShare configuration
    modelBuilder.Entity<DocumentShare>(entity =>
    {
        entity.HasKey(e => e.DocumentShareId);
        entity.HasIndex(e => new { e.DocumentId, e.UserId }).IsUnique();
        entity.HasOne(e => e.Document).WithMany().HasForeignKey(e => e.DocumentId).OnDelete(DeleteBehavior.Cascade);
    });

    // DocumentAuditLog configuration
    modelBuilder.Entity<DocumentAuditLog>(entity =>
    {
        entity.HasKey(e => e.AuditLogId);
        entity.Property(e => e.ActionType).IsRequired().HasMaxLength(20);
    });
}
```

## Category Values (Predefined List)

| Category | Description |
|----------|-------------|
| Project Documents | Documents related to specific projects |
| Team Resources | Shared resources for team use |
| Personal Files | Individual work files |
| Reports | Generated or authored reports |
| Presentations | Slide decks and presentation materials |
| Other | Miscellaneous documents not fitting other categories |

## File Type Allowlist

| MIME Type | Extension(s) | Category |
|-----------|-------------|----------|
| application/pdf | .pdf | Document |
| application/msword | .doc | Office |
| application/vnd.openxmlformats-officedocument.wordprocessingml.document | .docx | Office |
| application/vnd.ms-excel | .xls | Office |
| application/vnd.openxmlformats-officedocument.spreadsheetml.sheet | .xlsx | Office |
| application/vnd.ms-powerpoint | .ppt | Office |
| application/vnd.openxmlformats-officedocument.presentationml.presentation | .pptx | Office |
| text/plain | .txt | Text |
| image/jpeg | .jpg, .jpeg | Image |
| image/png | .png | Image |
