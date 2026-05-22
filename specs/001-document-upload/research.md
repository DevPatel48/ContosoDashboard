# Research: Document Upload and Management

## Blazor Server File Upload Patterns

### Decision: Use `InputFile` component with `EditForm` for file uploads

**Rationale**: Blazor Server provides the `InputFile` component (`<InputFile OnChange="OnFileSelected" />`) which exposes `FileReference` objects containing the uploaded file data. This is the recommended Microsoft approach for Blazor file uploads.

**Key Implementation Details**:
- `InputFile` component supports `multiple` attribute for multi-file selection
- `FileReference.OpenReadStream()` provides streaming access to file content
- Progress tracking can be achieved via `IJSRuntime` interop with JavaScript progress events, or by using chunked reads with Blazor's built-in progress reporting
- For a training system with 50 concurrent uploads, streaming is essential to avoid memory pressure

**Alternatives Considered**:
- **Multipart form submission**: Traditional HTML form approach; less Blazor-idiomatic, requires full page post
- **JavaScript interop file reader**: More complex, requires custom JS code
- **SignalR file transfer**: Overly complex for this use case; `InputFile` handles the streaming natively

**References**:
- Microsoft Docs: [File uploads in Blazor](https://learn.microsoft.com/en-us/aspnet/core/blazor/file-uploads)
- Maximum request body size: Configure via `builder.Services.AddMvc(options => options.MaxRequestBodySize = 25 * 1024 * 1024)` (25 MB)

## EF Core File Storage Pattern

### Decision: Store file paths as strings in database; files reside on local filesystem

**Rationale**: The stakeholder document explicitly requires local filesystem storage outside `wwwroot`. The database stores only metadata (Document entity) with a `FilePath` property pointing to the GUID-based storage location.

**Key Implementation Details**:
- `Document.FilePath` stores the relative path from the storage root (e.g., `documents/a1b2c3d4-e5f6-7890.pdf`)
- `IFileStorageService` abstracts filesystem operations (save, retrieve, delete, exists)
- Upload sequence: generate GUID path → save file to disk → save metadata to database (prevents orphaned records)
- Delete sequence: delete database record → delete file from disk (or vice versa with transaction)

**Alternatives Considered**:
- **Database BLOB storage**: Would store files in SQL Server; violates stakeholder requirement for local filesystem
- **Embedded files in wwwroot**: Security risk; files would be web-accessible directly
- **Cloud storage (Azure Blob)**: Violates offline-first principle

## File Storage Security

### Decision: Store files in `%LOCALAPPDATA%\ContosoDashboard\Documents\` with GUID-based filenames

**Rationale**: Files must be stored outside `wwwroot` (web-accessible directory) to prevent direct URL access. Using `%LOCALAPPDATA%` provides a standard Windows location with appropriate filesystem permissions.

**Key Implementation Details**:
- Storage root: `Environment.GetFolderPath(Environment.SpecialFolder.LocalApplicationData) / "ContosoDashboard" / "Documents"`
- Filename format: `{GUID}.{originalExtension}` (e.g., `a1b2c3d4-e5f6-7890-1234-567890abcdef.pdf`)
- Path traversal prevention: Validate that generated paths stay within storage root using `Path.GetFullPath()` and `StartsWith()` check
- Directory creation: Create storage root on first upload with `Directory.CreateDirectory()`
- File existence check: Verify file exists before serving; handle "orphaned record" case (DB record exists, file missing)

**Security Controls**:
- No direct URL access to files (outside wwwroot)
- GUID filenames prevent enumeration attacks
- Path traversal validation on all file operations
- Filesystem permissions on storage directory (read/write for application, no execute)

## Search Implementation

### Decision: EF Core LINQ `Contains` search with indexed columns

**Rationale**: For a training system with 10,000 documents maximum, LINQ-based text search is sufficient and avoids the complexity of full-text search indexes or external search engines.

**Key Implementation Details**:
- Search query: `Where(d => d.Title.Contains(searchTerm) || d.Description.Contains(searchTerm) || d.Tags.Contains(searchTerm))`
- EF Core translates `Contains` to SQL `LIKE` with wildcards
- For 2-second response time on 10K documents, add database indexes on searchable columns
- Consider `EF.Functions.Like()` for more flexible pattern matching if needed

**Alternatives Considered**:
- **EF Core Full-Text Search**: Requires SQL Server FTS configuration; overkill for 10K documents
- **Elasticsearch/Lucene**: External dependency; violates offline-first principle
- **In-memory filtering**: Would load all documents into memory; poor scalability

**Index Strategy**:
- `Document.Title`: Standard index
- `Document.Category`: Standard index (for filtering)
- `Document.UploadedBy`: Foreign key index
- `Document.AssociatedProjectId`: Foreign key index
- Consider computed column for concatenated search fields if query performance requires it

## Blazor Component Architecture

### Decision: Reusable `DocumentUpload` component embedded in multiple pages

**Rationale**: Document upload functionality is needed on multiple pages (Documents page, Task detail page, Project detail page). A reusable component avoids code duplication and ensures consistent UX.

**Key Implementation Details**:
- Component parameters: `AssociatedProjectId` (optional), `OnUploadComplete` callback
- Internal state: File selection, metadata form (title, category, description, tags)
- Validation: Client-side validation for required fields before upload
- Event-driven: Fires `UploadComplete` event when upload succeeds, allowing parent page to refresh

**Component Structure**:
```
DocumentUpload.razor
├── File selection area (InputFile component)
├── Metadata form (title, category, description, tags)
├── Upload progress indicator
└── OnUploadComplete callback to parent
```

**Integration Points**:
- `Documents.razor`: Primary upload location
- `Tasks.razor` (task detail): Upload with automatic project association
- `ProjectDetails.razor`: Upload with project pre-selected

## Service Layer Design

### Decision: Repository-free service layer with direct EF Core context access

**Rationale**: For a training system with 100 users and straightforward CRUD operations, a repository pattern adds unnecessary abstraction. Direct EF Core usage in services is cleaner and more idiomatic for this scale.

**Key Implementation Details**:
- `DocumentService` depends on `ApplicationDbContext` + `IFileStorageService`
- Service methods perform authorization checks before data operations
- Unit of work pattern: Single DbContext per request handles transaction scope
- No CQRS, no mediator pattern (overkill for this scale)

**Service Interface**:
```csharp
interface IDocumentService {
    Task<Document> UploadAsync(DocumentUploadRequest request);
    Task<IEnumerable<Document>> GetMyDocumentsAsync(string userId);
    Task<IEnumerable<Document>> GetProjectDocumentsAsync(int projectId);
    Task<IEnumerable<Document>> GetSharedDocumentsAsync(string userId);
    Task<Document> GetDocumentAsync(int documentId, string userId);
    Task ShareDocumentAsync(int documentId, string userId, string sharedWithUserId);
    Task RevokeShareAsync(int documentId, string sharedWithUserId);
    Task DeleteDocumentAsync(int documentId, string userId);
    Task UpdateMetadataAsync(int documentId, string userId, MetadataUpdate update);
    Task<IEnumerable<Document>> SearchAsync(string userId, string searchTerm);
}
```

## Notification Integration

### Decision: Reuse existing `NotificationService` for document sharing notifications

**Rationale**: The existing `NotificationService` provides in-app notifications. Document sharing notifications can leverage this service without introducing new notification infrastructure.

**Key Implementation Details**:
- On share: Call `NotificationService.SendNotificationAsync(sharedWithUserId, "Document shared: {title}", NotificationType.DocumentShared)`
- On project document upload: Notify project members via existing notification system
- Notification payload includes document ID for direct navigation

## Summary of Technical Decisions

| Area | Decision | Rationale |
|------|----------|-----------|
| File Upload | `InputFile` component + streaming | Blazor-idiomatic, supports progress |
| File Storage | Local filesystem with GUID filenames | Stakeholder requirement, security |
| Storage Path | `%LOCALAPPDATA%\ContosoDashboard\Documents\` | Outside wwwroot, standard Windows location |
| Search | EF Core LINQ `Contains` + indexes | Sufficient for 10K documents, offline-first |
| Components | Reusable `DocumentUpload` component | DRY principle, multiple integration points |
| Services | Direct EF Core in services (no repository) | Simpler architecture for training scale |
| Notifications | Reuse existing `NotificationService` | Leverages existing infrastructure |
| Virus Scanning | Simulated scanner interface | Training demonstration, placeholder for future |
