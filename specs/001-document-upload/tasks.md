# Tasks: Document Upload and Management

**Feature**: Document Upload and Management  
**Branch**: `001-document-upload`  
**Spec**: `specs/001-document-upload/spec.md`  
**Plan**: `specs/001-document-upload/plan.md`

## Format: `[ID] [P?] [Story?] Description with file path`

- **[P]**: Can run in parallel (different files, no dependencies)
- **[Story]**: Which user story this task belongs to (e.g., US1, US2, US3)
- Include exact file paths in descriptions

## Path Conventions

All paths are relative to `ContosoDashboard/` project root.

---

## Phase 1: Setup (Shared Infrastructure)

**Purpose**: Project initialization and configuration for document storage

- [ ] T001 Configure maximum request body size (25 MB) in `Program.cs`
- [ ] T002 [P] Add document storage configuration to `appsettings.json` and `appsettings.Development.json`

---

## Phase 2: Foundational (Blocking Prerequisites)

**Purpose**: Core models, database schema, and service interfaces that ALL user stories depend on

**⚠️ CRITICAL**: No user story work can begin until this phase is complete

- [ ] T003 Create Document model in `Models/Document.cs`
- [ ] T004 [P] Create DocumentShare model in `Models/DocumentShare.cs`
- [ ] T005 [P] Create DocumentAuditLog model in `Models/DocumentAuditLog.cs`
- [ ] T006 Extend ApplicationDbContext with DbSet properties and OnModelCreating configuration in `Data/ApplicationDbContext.cs`
- [ ] T007 Create EF Core migration `AddDocumentEntities` and update database schema
- [ ] T008 [P] Create IFileStorageService interface in `Services/IFileStorageService.cs`
- [ ] T009 [P] Create IDocumentService interface in `Services/IDocumentService.cs`
- [ ] T010 Implement FileStorageService with GUID filenames, path traversal prevention, and storage root management in `Services/FileStorageService.cs`
- [ ] T011 Register IFileStorageService and IDocumentService as scoped services in `Program.cs`

**Checkpoint**: Foundation ready — models, database, and service interfaces are in place for user story implementation

---

## Phase 3: User Story 1 - Upload Work Documents (Priority: P1) 🎯 MVP

**Goal**: Users can upload one or more files with metadata (title, category, optional description/tags/project), with validation for file type, size, and required fields.

**Independent Test**: Login as employee → navigate to Documents page → select valid PDF under 25 MB → enter title and category → verify file appears in "My Documents" with correct metadata.

### Implementation for User Story 1

- [ ] T012 [US1] Implement DocumentService.UploadAsync with file validation, GUID path generation, and audit logging in `Services/DocumentService.cs`
- [ ] T013 [US1] Implement DocumentService.GetMyDocumentsAsync with authorization check in `Services/DocumentService.cs`
- [ ] T014 [P] Create DocumentUpload reusable component with InputFile, metadata form, validation, and progress indicator in `Shared/DocumentUpload.razor`
- [ ] T015 [US1] Create Documents.razor page with My Documents list view, search bar, category filter, and embedded DocumentUpload component in `Pages/Documents.razor`
- [ ] T016 [US1] Implement empty state display ("No documents yet. Upload your first document!") with Upload button in `Pages/Documents.razor`
- [ ] T017 [US1] Add file type allowlist validation and file size limit enforcement (25 MB) in `Services/DocumentService.cs`
- [ ] T018 [US1] Add simulated virus scanning interface with placeholder implementation in `Services/DocumentService.cs`

**Checkpoint**: At this point, users can upload documents and see them in "My Documents" — the core MVP is functional

---

## Phase 4: User Story 2 - Browse and Search Documents (Priority: P2)

**Goal**: Users can browse their documents with sorting, filtering by category/project, and search across title, description, and tags.

**Independent Test**: Upload several documents with different categories → filter by category → sort by date → search by title → verify correct results appear within 2 seconds.

### Implementation for User Story 2

- [ ] T019 [P] [US2] Implement DocumentService.SearchAsync with LINQ Contains across title, description, and tags in `Services/DocumentService.cs`
- [ ] T020 [P] [US2] Implement DocumentService.GetProjectDocumentsAsync with project association filter in `Services/DocumentService.cs`
- [ ] T021 [US2] Add sorting controls (title, date, category, size) to Documents.razor in `Pages/Documents.razor`
- [ ] T022 [US2] Add category filter dropdown and project filter dropdown to Documents.razor in `Pages/Documents.razor`
- [ ] T023 [US2] Add search input with debounced search to Documents.razor in `Pages/Documents.razor`
- [ ] T024 [US2] Create Project Documents view showing documents for a specific project with empty state in `Pages/Documents.razor`

**Checkpoint**: Users can now upload, browse, filter, and search their documents

---

## Phase 5: User Story 3 - Share Documents with Team Members (Priority: P3)

**Goal**: Document owners can share documents with specific users; recipients receive notifications and see shared documents in "Shared with Me."

**Independent Test**: User A shares document with User B → login as User B → verify notification appears → verify document shows in "Shared with Me" → User A revokes share → verify document disappears from User B's view.

### Implementation for User Story 3

- [ ] T025 [P] [US3] Implement DocumentService.ShareDocumentAsync with duplicate share prevention in `Services/DocumentService.cs`
- [ ] T026 [P] [US3] Implement DocumentService.RevokeShareAsync in `Services/DocumentService.cs`
- [ ] T027 [P] [US3] Implement DocumentService.GetSharedDocumentsAsync in `Services/DocumentService.cs`
- [ ] T028 [US3] Create SharedDocuments.razor page with "Shared with Me" list view and empty state in `Pages/SharedDocuments.razor`
- [ ] T029 [US3] Create DocumentDetails.razor page with metadata display, share modal, and user list for sharing in `Pages/DocumentDetails.razor`
- [ ] T030 [US3] Integrate NotificationService for share/unshare notifications in `Services/DocumentService.cs`
- [ ] T031 [US3] Add navigation link for "Shared Documents" to `Shared/NavMenu.razor`

**Checkpoint**: Users can share documents with teammates and access shared documents

---

## Phase 6: User Story 4 - Download and Preview Documents (Priority: P3)

**Goal**: Users can download any document they have access to and preview PDF/image files in-browser.

**Independent Test**: Upload PDF and image → click download → verify file downloads with correct name → click preview → verify PDF/image renders in browser.

### Implementation for User Story 4

- [ ] T032 [P] [US4] Implement document download endpoint with authorization check and file stream in `Pages/DocumentDetails.razor`
- [ ] T033 [P] [US4] Create file download handler using FileStorageService.GetFileAsync in `Services/DocumentService.cs`
- [ ] T034 [US4] Add in-browser preview for PDF (iframe/embed) and images in `Pages/DocumentDetails.razor`
- [ ] T035 [US4] Add IDOR protection: DocumentService.GetDocumentAsync authorization validation in `Services/DocumentService.cs`
- [ ] T036 [US4] Handle missing file scenario (DB record exists, file missing) with error display in `Services/DocumentService.cs`

**Checkpoint**: Users can download and preview documents they have access to

---

## Phase 7: User Story 5 - Manage Document Metadata (Priority: P4)

**Goal**: Document owners can edit title, description, category, tags, or replace the file with an updated version.

**Independent Test**: Edit document title and save → verify updated title appears in list → replace file → verify new content stored and metadata preserved.

### Implementation for User Story 5

- [ ] T037 [P] [US5] Implement DocumentService.UpdateMetadataAsync with ownership validation in `Services/DocumentService.cs`
- [ ] T038 [P] [US5] Implement DocumentService.ReplaceFileAsync with old file cleanup in `Services/DocumentService.cs`
- [ ] T039 [US5] Add metadata edit form (title, description, category, tags) to DocumentDetails.razor in `Pages/DocumentDetails.razor`
- [ ] T040 [US5] Add file replace functionality to DocumentDetails.razor in `Pages/DocumentDetails.razor`

**Checkpoint**: Users can update document metadata and replace file content

---

## Phase 8: User Story 6 - Delete Documents (Priority: P4)

**Goal**: Users can delete their own documents with confirmation. Project Managers can delete any document in their projects.

**Independent Test**: Upload document → delete with confirmation → verify document removed from list and file deleted from disk.

### Implementation for User Story 6

- [ ] T041 [P] [US6] Implement DocumentService.DeleteDocumentAsync with ownership/role validation in `Services/DocumentService.cs`
- [ ] T042 [P] [US6] Implement file deletion from storage on document delete in `Services/DocumentService.cs`
- [ ] T043 [US6] Add delete confirmation dialog to Documents.razor in `Pages/Documents.razor`
- [ ] T044 [US6] Add delete button to DocumentDetails.razor with role-based visibility (owner or PM) in `Pages/DocumentDetails.razor`

**Checkpoint**: Users can delete their documents; PMs can delete project documents

---

## Phase 9: Polish & Cross-Cutting Concerns

**Purpose**: Navigation integration, dashboard widgets, audit logging, and final refinements

- [ ] T045 Add "Documents" navigation link to `Shared/NavMenu.razor`
- [ ] T046 [P] Add "Recent Documents" widget (last 5 uploads) to dashboard in `Pages/Index.razor`
- [ ] T047 [P] Add document count to dashboard summary cards in `Pages/Index.razor`
- [ ] T048 Add DocumentUpload component to project detail page with project pre-selected in `Pages/ProjectDetails.razor`
- [ ] T049 Add DocumentUpload component to task detail page with automatic project association in `Pages/Tasks.razor`
- [ ] T050 Implement DocumentService.GetAuditLogAsync and display audit trail in `Pages/DocumentDetails.razor`
- [ ] T051 [P] Add audit log entries for all document operations (upload, download, share, delete, edit, replace) in `Services/DocumentService.cs`
- [ ] T052 [P] Add role-based access controls for Administrators, ProjectManagers, TeamLeads, and Employees in `Services/DocumentService.cs`
- [ ] T053 Apply Bootstrap 5.3 styling to all document pages for consistent UI in `Pages/Documents.razor`, `Pages/DocumentDetails.razor`, `Pages/SharedDocuments.razor`
- [ ] T054 Add error handling and user-friendly error messages for all document operations in `Services/DocumentService.cs`

---

## Dependency Graph

```
Phase 1 (Setup)
    ↓
Phase 2 (Foundational) — BLOCKING for all user stories
    ↓
Phase 3 (US1-P1: Upload) — MVP core
    ↓
Phase 4 (US2-P2: Browse/Search) — depends on US1 for document data
    ↓
Phase 5 (US3-P3: Sharing) — depends on US1+US2 for documents and users
    ↓
Phase 6 (US4-P3: Download/Preview) — depends on US1 for documents
    ↓
Phase 7 (US5-P4: Metadata) — depends on US1 for documents
    ↓
Phase 8 (US6-P4: Delete) — depends on US1 for documents
    ↓
Phase 9 (Polish) — depends on all phases for integration
```

## Parallel Execution Examples

### Phase 2 (Foundational)
- T003, T004, T005: Models can be created in parallel (independent files)
- T008, T009: Service interfaces can be created in parallel (independent files)
- T010: FileStorageService depends on T008 (interface) but can proceed after
- T006: ApplicationDbContext extension depends on T003-T005 (models)

### Phase 3 (US1 - Upload)
- T014: DocumentUpload component can be developed in parallel with T012 (service)
- T015: Documents.razor depends on T012 (GetMyDocuments) and T014 (upload component)

### Phase 9 (Polish)
- T046, T047: Dashboard widgets can be developed in parallel
- T048, T049: Page integrations can be developed in parallel
- T051, T052: Audit logging and RBAC can be developed in parallel

---

## Task Summary

| Phase | Task Count | Description |
|-------|-----------|-------------|
| Phase 1: Setup | 2 | Project configuration |
| Phase 2: Foundational | 9 | Models, DbContext, interfaces, service registration |
| Phase 3: US1-P1 (Upload) | 7 | Core MVP — upload and My Documents |
| Phase 4: US2-P2 (Browse/Search) | 6 | Filtering, sorting, search, project view |
| Phase 5: US3-P3 (Sharing) | 7 | Share/revoke, SharedDocuments, notifications |
| Phase 6: US4-P3 (Download/Preview) | 5 | Download, preview, IDOR protection |
| Phase 7: US5-P4 (Metadata) | 4 | Edit metadata, replace file |
| Phase 8: US6-P4 (Delete) | 4 | Delete with confirmation, PM delete |
| Phase 9: Polish | 10 | Navigation, dashboard, audit, RBAC, styling |
| **Total** | **54** | **Complete feature implementation** |

## MVP Scope

**Suggested MVP**: Phase 1 + Phase 2 + Phase 3 (US1-P1) = 18 tasks (T001-T018)

This delivers the core capability: users can upload documents with metadata and see them in "My Documents." All subsequent phases add incremental value on top of this foundation.
