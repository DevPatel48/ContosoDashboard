// Service Interface: IFileStorageService
// Defines the contract for file storage operations on the local filesystem

namespace ContosoDashboard.Services.Contracts;

public interface IFileStorageService
{
    /// <summary>
    /// Save a file stream to the storage directory with a GUID-based filename.
    /// Returns the relative file path that should be stored in the database.
    /// </summary>
    Task<string> SaveFileAsync(Stream content, string originalFileName);

    /// <summary>
    /// Get a file stream for reading (download).
    /// </summary>
    Task<(Stream stream, string? fileName)> GetFileAsync(string filePath);

    /// <summary>
    /// Delete a file from storage.
    /// </summary>
    Task DeleteFileAsync(string filePath);

    /// <summary>
    /// Check if a file exists in storage.
    /// </summary>
    Task<bool> FileExistsAsync(string filePath);

    /// <summary>
    /// Replace an existing file with new content.
    /// </summary>
    Task<string> ReplaceFileAsync(string oldFilePath, Stream newContent, string newFileName);

    /// <summary>
    /// Get the storage root directory path.
    /// </summary>
    string GetStorageRoot();
}
