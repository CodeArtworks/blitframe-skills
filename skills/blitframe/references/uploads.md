# Large files and ZIPs

For a file too large to send inline in an edit (the request is at most 64 MB, about 45 MB of binary once base64-encoded), a new file for a render's replacements, or a whole project as a ZIP. MCP tools are written `blitframe:<tool>`.

## Upload, then use

1. **Create the upload** with the file's real name and exact size: `blitframe:create_upload` (`POST /uploads` with `fileName` and `sizeBytes`). The extension must be a supported file type, or `.zip`.
2. **`PUT` the raw bytes** to the returned `url`, without an `Authorization` header: `curl -X PUT --upload-file background.mp4 "$UPLOAD_URL"`. The URL accepts exactly `sizeBytes` bytes.
3. **Complete it:** `blitframe:complete_upload` (`POST /uploads/{uploadId}/complete`). Blitframe then checks the file. While `status` is `processing`, wait with `blitframe:get_upload` (`GET /uploads/{uploadId}` with `Prefer: wait=50`) until it is `ready` or `failed`.
4. **Use the `ready` upload once:**
   - in an edit, as a file: `{ "path": "video/background.mp4", "uploadId": "..." }`, at a path with the upload's extension;
   - in a replacement, as a new image or video: `"file": { "path": "images/product-42.png", "uploadId": "..." }`, in an edit or a render (a render takes new files only this way);
   - as a new project: `blitframe:create_project` (`POST /projects`) with `name` and `zipUploadId`;
   - as a project's next version: `blitframe:import_zip` (`POST /projects/{projectId}/package`) with `zipUploadId`, `variation` to import onto a variation other than `main`, and `expectedHeadVersionId` when it must replace the head you read.
5. **Send an idempotency key** with the edit, render, `blitframe:create_project` or `blitframe:import_zip` that uses the upload (`idempotencyKey` over MCP, the `Idempotency-Key` header over REST). Each upload is used once, so a retry without a key after a lost answer could fail on the used upload; with the same key it answers with what the first call made. An edit that names uploads is safe to send again even without a key: the same edit returns the version it made.

## Above 100 MB

`blitframe:create_upload` (`POST /uploads`) answers with `multipart.parts` instead of `url`: one URL per part of `partSize` bytes, the last part holding the rest.

1. `PUT` each part to its URL and keep each response's `ETag` header. Parts can go in parallel.
2. Complete with every part: `{ "parts": [{ "partNumber": 1, "etag": "..." }, ...] }`.
3. Part URLs last 1 hour. `blitframe:get_upload_part_urls` (`POST /uploads/{uploadId}/parts`) issues fresh ones to resume.

## Rules

- A failed upload's `error.code` says what to do: `interrupted` means complete it again (the bytes are not sent again); anything else is about the file, so create a new upload.
- An edit or render that names an upload answers `upload_incomplete`, `upload_processing` or `upload_failed` until the upload is `ready`. Each upload is used once: a batch of renders needs one upload per row that uses it, and a render's uploads are used up once its version is made. To render those replacements again (after a frames preview, or for a failed row), render that `versionId` without replacements, or upload the files again.
- An unused upload expires after 1 hour (24 hours for multipart). A workspace holds at most 100 unused uploads (`upload_limit`); `blitframe:delete_upload` (`DELETE /uploads/{uploadId}`) discards one, after asking the user.
- Uploads go up to 8 GB, but each file type has its own limit (128 MB for an image, 4 GB for a video), and the plan caps a project: the files of one version together (`limits.maxProjectBytes` in `blitframe:get_usage`). An upload, edit or ZIP import past it is `403 project_too_large` (a ZIP's upload only when it is too large to import over any head): tell the user to remove files or choose a larger plan, and don't retry. `blitframe:get_file_capabilities` (`GET /files/capabilities`) lists every type with its limits.
- A ZIP has `data.json` at its root, or inside one top-level folder. A ZIP with font files needs `fontLicenseAccepted: true`. A ZIP import replaces every file of the project with the ZIP's; an edit changes only the files it names.
- A large ZIP still importing after about 50 seconds answers with an import instead of a version: follow it with `blitframe:get_zip_import` (`GET /projects/{projectId}/package/{importId}` with `Prefer: wait=50`) until `status` is `succeeded` or `failed`.
- A file already in any version of the workspace needs no upload: an edit's file `{ "path": "...", "fromVersionId": "...", "fromPath": "..." }`. `blitframe:list_project_files` (`GET /projects/{projectId}/files`) finds one.
- If your sandbox cannot make HTTP `PUT` requests, send files inline in an edit with `contentBase64` instead. For a render, add the file to the project that way first and replace by `path`.

Every detail: `https://codeartworks.com/docs/blitframe/uploads.md`.
