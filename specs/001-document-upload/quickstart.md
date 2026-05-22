# Quickstart: Document Upload and Management Implementation

## Prerequisites

- .NET 8.0 SDK installed
- Visual Studio 2022 or VS Code with C# Dev Kit
- SQL Server LocalDB (included with Visual Studio)
- Existing ContosoDashboard project structure

## Implementation Order

Follow this sequence to implement the feature incrementally:

### Step 1: Models and Database Migration

1. Create model files:
   - `Models/Document.cs`
   - `Models/DocumentShare.cs`
   - `Models/DocumentAuditLog.cs`

2. Update `ApplicationDbContext.cs`:
   - Add `DbSet<Document>`, `DbSet<DocumentShare>`, `DbSet<DocumentAuditLog>`
   - Configure entity relationships in `OnModelCreating()`

3. Create and apply migration:
   ```bash
   dotnet ef migrations add AddDocumentEntities
   dotnet ef database update
   ```

### Step 2: File Storage Service

1. Create `Services/IFileStorageService.cs` interface
2. Create `Services/FileStorageService.cs` implementation:
   - Storage root: `%LOCALAPPDATA%\ContosoDashboard\Documents\`
   - GUID-based filenames
   - Path traversal validation
   - Stream-based file operations

3. Register in `Program.cs`:
   ```csharp
   builder.Services.AddScoped<IFileStorageService, FileStorageService>();
   ```

### Step 3: Document Service

1. Create `Services/IDocumentService.cs` interface
2. Create `Services/DocumentService.cs` implementation:
   - Depends on `ApplicationDbContext` + `IFileStorageService`
   - Authorization checks in each method
   - Audit log creation on all operations

3. Register in `Program.cs`:
   ```csharp
   builder.Services.AddScoped<IDocumentService, DocumentService>();
   ```

### Step 4: Blazor Components

1. Create `Shared/DocumentUpload.razor`:
   - File selection (`InputFile` component)
   - Metadata form (title, category, description, tags)
   - Upload progress indicator
   - `OnUploadComplete` callback

2. Create `Pages/Documents.razor`:
   - My Documents view (table/list)
   - Shared with Me view
   - Search bar
   - Category filter
   - DocumentUpload component embedded

3. Create `Pages/DocumentDetails.razor`:
   - Document metadata display
   - Download button
   - Share modal
   - Audit log display

### Step 5: Navigation Integration

1. Update `Shared/NavMenu.razor`:
   - Add "Documents" navigation link
   - Show based on authenticated user

2. Update `Pages/ProjectDetails.razor`:
   - Add "Documents" tab with DocumentUpload component
   - Show project-associated documents

3. Update `Pages/Tasks.razor`:
   - Add document upload option on task detail view

### Step 6: Testing

1. Unit tests (create `Tests/Unit/`):
   - `DocumentServiceTests.cs`: Test service methods with mock DbContext
   - `FileStorageServiceTests.cs`: Test file operations with temp directory

2. Integration tests (create `Tests/Integration/`):
   - `DocumentUploadIntegrationTests.cs`: Test full upload flow
   - `DocumentSearchIntegrationTests.cs`: Test search functionality

### Step 7: Configuration

1. Update `appsettings.json`:
   ```json
   {
     "DocumentStorage": {
       "StorageRoot": "%LOCALAPPDATA%\\ContosoDashboard\\Documents",
       "MaxFileSizeBytes": 26214400
     }
   }
   ```

2. Configure maximum request size in `Program.cs`:
   ```csharp
   builder.Services.AddMvc(options => 
       options.MaxRequestBodySize = 25 * 1024 * 1024);
   ```

## Build and Run

```bash
# Build the project
dotnet build

# Run the application
dotnet run

# Navigate to http://localhost:5000 (or assigned port)
```

## Verification Checklist

- [ ] Can upload a document with metadata
- [ ] File is stored in `%LOCALAPPDATA%\ContosoDashboard\Documents\`
- [ ] Database record created with correct metadata
- [ ] "My Documents" view shows uploaded document
- [ ] Can search and find document by title
- [ ] Can share document with another user
- [ ] "Shared with Me" view shows shared document
- [ ] Can download document
- [ ] Can delete document (removes DB record + file)
- [ ] Audit log records all actions
- [ ] File size limit enforced (25 MB)
- [ ] File type validation works
- [ ] Category dropdown shows predefined values

## Common Issues and Solutions

### Issue: "The file is too large"
**Solution**: Increase `MaxRequestBodySize` in `Program.cs` or reduce file size.

### Issue: "Access to path denied"
**Solution**: Ensure storage root directory exists and has correct permissions. The application will create it on first upload.

### Issue: "Migration already exists"
**Solution**: Run `dotnet ef migrations remove` to remove the migration, then recreate it.

### Issue: "File not found" after upload
**Solution**: Check that `IFileStorageService.SaveFileAsync()` is being called before database save. Verify the storage root path is correct.

## Next Steps After Implementation

1. Run `/speckit.implement` to execute the task list
2. Perform manual testing on all acceptance criteria
3. Code review for security and performance
4. Deploy to training environment
5. User acceptance testing
