// Service Interface: IDocumentService
// Defines the contract for document management operations

namespace ContosoDashboard.Services.Contracts;

public interface IDocumentService
{
    /// <summary>
    /// Upload a new document with metadata and file content.
    /// </summary>
    Task<Document> UploadAsync(DocumentUploadRequest request);

    /// <summary>
    /// Get all documents uploaded by the current user.
    /// </summary>
    Task<IEnumerable<Document>> GetMyDocumentsAsync(string userId);

    /// <summary>
    /// Get all documents associated with a specific project.
    /// </summary>
    Task<IEnumerable<Document>> GetProjectDocumentsAsync(int projectId);

    /// <summary>
    /// Get all documents shared with the current user.
    /// </summary>
    Task<IEnumerable<Document>> GetSharedDocumentsAsync(string userId);

    /// <summary>
    /// Get a specific document by ID (with authorization check).
    /// </summary>
    Task<Document?> GetDocumentAsync(int documentId, string userId);

    /// <summary>
    /// Share a document with another user.
    /// </summary>
    Task ShareDocumentAsync(int documentId, string userId, string sharedWithUserId, string accessLevel);

    /// <summary>
    /// Revoke a document share.
    /// </summary>
    Task RevokeShareAsync(int documentId, string userId, string sharedWithUserId);

    /// <summary>
    /// Delete a document (soft delete).
    /// </summary>
    Task DeleteDocumentAsync(int documentId, string userId);

    /// <summary>
    /// Update document metadata.
    /// </summary>
    Task UpdateMetadataAsync(int documentId, string userId, MetadataUpdate update);

    /// <summary>
    /// Search documents by title, description, or tags.
    /// </summary>
    Task<IEnumerable<Document>> SearchAsync(string userId, string searchTerm);

    /// <summary>
    /// Get audit log for a specific document.
    /// </summary>
    Task<IEnumerable<DocumentAuditLog>> GetAuditLogAsync(int documentId, string userId);

    /// <summary>
    /// Replace document file content.
    /// </summary>
    Task ReplaceFileAsync(int documentId, string userId, Stream newFile, string newFileName, string newFileType);
}

public class DocumentUploadRequest
{
    public required Stream FileContent { get; set; }
    public required string FileName { get; set; }
    public required string FileType { get; set; }
    public required string Title { get; set; }
    public required string Category { get; set; }
    public string? Description { get; set; }
    public string? Tags { get; set; }
    public int? AssociatedProjectId { get; set; }
    public required string UploadedBy { get; set; }
}

public class MetadataUpdate
{
    public string? Title { get; set; }
    public string? Description { get; set; }
    public string? Category { get; set; }
    public string? Tags { get; set; }
}
