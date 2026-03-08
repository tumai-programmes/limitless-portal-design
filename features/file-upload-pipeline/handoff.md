# File Upload Pipeline — Handoff Note

**Created:** 2026-03-08
**Origin chat:** [Wizard UI improvements](87bb45a0-185f-4f8b-8955-6c5a96e9fcfa)
**Status:** Ready for implementation

## Problem Statement

All file upload components in the Limitless Portal wizards are **placeholders**. When a user selects "Upload what we have" and picks a file, DevExtreme shows "Ready to upload" — but no actual upload ever happens. The file sits in the browser's memory with no server-side destination.

This affects three Vue components and every wizard instrument that uses them:

| Component | Question Type | Where Used |
|-----------|--------------|------------|
| `QFileUpload.vue` | `FileUpload` | All instruments |
| `QChecklistUpload.vue` | `ChecklistUpload` | Company Audit (Section B), others |
| `QResponseMode.vue` | `ResponseMode` | All instruments |

All three use `upload-mode="useForm"` on `DxFileUploader`, which stages files locally but waits for a form submission trigger that doesn't exist.

## What Already Exists (Backend)

The backend infrastructure is **partially built** but **not wired to any routes**.

### S3 Service — `internal/services/s3.go`

```go
type S3Client struct {
    client *s3.Client
    bucket string
}

func (s *S3Client) GeneratePresignedUploadURL(ctx, key, contentType, expiry) (string, error)
func (s *S3Client) GeneratePresignedDownloadURL(ctx, key, expiry) (string, error)
func StoragePath(engagementID, category, filename string) string
    // returns "engagements/{id}/{category}/{filename}"
```

### Upload Model — `internal/models/engagement.go`

```go
type Upload struct {
    ID           string    `json:"id"`
    EngagementID string    `json:"engagement_id"`
    SubmissionID *string   `json:"submission_id,omitempty"`
    UserID       string    `json:"user_id"`
    Filename     string    `json:"filename"`
    MimeType     string    `json:"mime_type"`
    SizeBytes    int64     `json:"size_bytes"`
    StoragePath  string    `json:"storage_path"`
    Section      *string   `json:"section,omitempty"`
    FieldKey     *string   `json:"field_key,omitempty"`
    UploadedAt   time.Time `json:"uploaded_at"`
}
```

### Repository Interface — `internal/storage/repository.go`

```go
CreateUpload(ctx context.Context, upload *models.Upload) error
ListUploads(ctx context.Context, engagementID string) ([]models.Upload, error)
```

### Config — `internal/config/config.go`

AWS S3 env vars already defined and loaded:

| Variable | Default |
|----------|---------|
| `AWS_S3_BUCKET` | (required) |
| `AWS_S3_REGION` | `eu-west-2` |
| `AWS_ACCESS_KEY_ID` | (required) |
| `AWS_SECRET_ACCESS_KEY` | (required) |

### Database Schema — `migrations/001_initial_schema.up.sql`

The `uploads` table already exists in the initial migration.

## What Does NOT Exist

| Missing Piece | Layer |
|--------------|-------|
| Upload handler (`internal/handlers/upload.go`) | Backend |
| Routes for presign + confirm endpoints | Backend |
| Frontend service/composable for S3 upload | Frontend |
| Upload progress bar / success feedback | Frontend |
| DxFileUploader wiring to presigned URL flow | Frontend |
| File list display (uploaded files per question) | Frontend |
| Delete/replace uploaded file | Frontend |

## Intended Architecture

Per the portal spec, the upload flow should be:

```
1. User selects file in DxFileUploader
2. Frontend calls: POST /api/uploads/presign
   Body: { engagementID, submissionID, section, fieldKey, filename, contentType, sizeBytes }
   Response: { uploadURL, storagePath, uploadID }

3. Frontend uploads file DIRECTLY to S3 via PUT to presigned URL
   (browser → S3, bypasses Go backend for the actual bytes)

4. Frontend calls: POST /api/uploads/confirm
   Body: { uploadID }
   Response: { upload object }

5. Upload record saved to database, file metadata stored in wizard answers
```

This keeps the Go backend lightweight (only generates URLs and records metadata) while S3 handles the heavy lifting of file transfer.

## Affected Frontend Components

### `QFileUpload.vue` (standalone file upload question)
- **Path:** `frontend/src/components/questions/QFileUpload.vue`
- Currently captures file names only via `onFilesChanged`
- Needs: presign → upload → confirm → store S3 key in `modelValue`

### `QChecklistUpload.vue` (checklist with per-item upload)
- **Path:** `frontend/src/components/questions/QChecklistUpload.vue`
- Each checklist item has its own upload when mode is `'upload'`
- Needs: same presign flow, scoped per `item.key`

### `QResponseMode.vue` (single response mode selector)
- **Path:** `frontend/src/components/questions/QResponseMode.vue`
- Single upload when mode is `'upload'`
- Needs: same presign flow

## Action Plan

### Phase 1: Backend — Upload Handler + Routes

1. Create `internal/handlers/upload.go` with two endpoints:
   - `POST /api/uploads/presign` — generates pre-signed S3 URL, creates pending Upload record
   - `POST /api/uploads/confirm` — marks upload as confirmed after successful S3 PUT
   - `GET /api/uploads?engagement_id=X` — lists uploads for an engagement
   - `DELETE /api/uploads/{id}` — deletes upload record and S3 object
2. Register routes in `internal/router/router.go` (behind auth middleware)
3. Verify S3 client is initialised in `main.go` and passed to handler

### Phase 2: Frontend — Upload Service

1. Create `frontend/src/services/upload.ts` (or composable `useUpload`)
   - `requestPresignedUrl(params)` → calls presign endpoint
   - `uploadToS3(file, presignedUrl)` → PUT with progress tracking via XHR
   - `confirmUpload(uploadId)` → calls confirm endpoint
   - Exposes reactive state: `uploading`, `progress`, `error`, `uploadedFiles`

### Phase 3: Frontend — Wire Components

1. Update `QFileUpload.vue`:
   - Switch `upload-mode` to `"useButtons"` or handle via custom button
   - On file select → presign → upload → confirm → update `modelValue` with S3 metadata
   - Show progress bar and success/error state
   - Display list of uploaded files with delete option

2. Update `QChecklistUpload.vue`:
   - Same flow but scoped per checklist `item.key`
   - Each item's upload is independent

3. Update `QResponseMode.vue`:
   - Same flow for the upload mode option

### Phase 4: Polish

1. File size validation (client-side before upload)
2. File type validation (from `question.acceptedFileTypes`)
3. Upload cancellation
4. Retry on failure
5. Display uploaded files when returning to a previously answered question

## Environment Checklist

Before starting implementation, verify these are configured on the deployment VM (`.env`):

- [ ] `AWS_S3_BUCKET` is set and the S3 bucket exists
- [ ] `AWS_ACCESS_KEY_ID` and `AWS_SECRET_ACCESS_KEY` have `s3:PutObject`, `s3:GetObject`, `s3:DeleteObject` permissions
- [ ] S3 bucket has CORS configured to allow browser PUT from `portal.limitlessmodus.com` and `dev-portal.limitlessmodus.com`
- [ ] `AWS_S3_REGION` matches the bucket's region (`eu-west-2`)

## Key Files Reference

| File | Purpose |
|------|---------|
| `limitless-portal/internal/services/s3.go` | S3 client with presign methods |
| `limitless-portal/internal/models/engagement.go` | `Upload` struct definition |
| `limitless-portal/internal/storage/repository.go` | `CreateUpload`, `ListUploads` interface |
| `limitless-portal/internal/storage/postgres.go` | Repository implementation |
| `limitless-portal/internal/config/config.go` | AWS env var loading |
| `limitless-portal/internal/router/router.go` | Route registration (add upload routes here) |
| `limitless-portal/migrations/001_initial_schema.up.sql` | Uploads table DDL |
| `limitless-portal/frontend/src/components/questions/QFileUpload.vue` | Standalone upload component |
| `limitless-portal/frontend/src/components/questions/QChecklistUpload.vue` | Checklist + upload component |
| `limitless-portal/frontend/src/components/questions/QResponseMode.vue` | Response mode + upload component |

## Estimated Effort

| Phase | Estimate |
|-------|----------|
| Backend handler + routes | 1-2 hours |
| Frontend upload service | 1-2 hours |
| Wire 3 components | 2-3 hours |
| Polish + testing | 1-2 hours |
| **Total** | **5-9 hours** |
