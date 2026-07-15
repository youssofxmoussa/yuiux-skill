---
name: file-upload-download
description: Upload progress, drag-and-drop, large-file handling, and download UX errors and fixes for web UI.
---

# File Upload & Download

## Anti-pattern: No progress indication during upload/download
**What's wrong:** Clicking "Upload" produces a static, unchanging button state for the entire duration of a multi-second (or longer) transfer.
**Why it matters:** Indistinguishable from a frozen/broken interaction; users may re-click, navigate away thinking it failed, or lose confidence in the product.
**Fix:** Show real, granular progress wherever the transfer mechanism supports it (`XMLHttpRequest.upload.onprogress`, or a resumable upload library's progress events), including a percentage or progress bar, not just an indeterminate spinner for anything beyond a couple seconds.

```js
xhr.upload.addEventListener("progress", (e) => {
  const percent = Math.round((e.loaded / e.total) * 100);
  setProgress(percent);
});
```

## Anti-pattern: Drag-and-drop with no visual feedback on drag-over
**What's wrong:** A drop zone that accepts files but shows no visual change when a file is dragged over it.
**Why it matters:** Users can't tell whether they're actually over a valid drop target until they release and see what happens (or doesn't).
**Fix:** Visually highlight the drop zone (border color change, background tint) on `dragenter`/`dragover`, and revert on `dragleave`/`drop`.

## Anti-pattern: Drag-and-drop is the only way to upload, with no fallback
**What's wrong:** A drop zone with no visible "Browse files" / "Choose file" button — the only way in is dragging.
**Why it matters:** Drag-and-drop is genuinely difficult or impossible for some motor-impaired users and keyboard-only users, and plenty of users simply don't think to try it without a button prompting the alternative.
**Fix:** Always pair a drop zone with a clickable button/link that opens the native file picker as an equally first-class path.

## Anti-pattern: File type/size validated only after the full upload completes
**What's wrong:** A large file uploads completely over several seconds/minutes, only then to be rejected server-side for being the wrong type or too large.
**Why it matters:** Wastes the user's time and bandwidth for a failure that was knowable instantly.
**Fix:** Validate file type and size client-side immediately on selection/drop, before starting the transfer, with a clear message stating the actual limit ("Files must be under 25MB — this file is 40MB").

## Anti-pattern: Upload failure gives no retry option, forcing a full restart
**What's wrong:** A failed upload (network blip, timeout) requires the user to re-select the file and start from zero.
**Why it matters:** Especially costly for large files on unreliable connections — a single dropped packet shouldn't cost the entire transfer.
**Fix:** Offer a retry action that resumes or restarts the specific failed upload without requiring the user to re-navigate/re-select the file if it's still held in memory/state; for very large files, prefer a chunked/resumable upload approach that can continue from where it left off.

## Anti-pattern: No way to cancel an in-progress upload
**What's wrong:** Once started, an upload can't be stopped short of closing the tab.
**Why it matters:** Users who selected the wrong file or changed their mind are stuck waiting out a transfer they no longer want.
**Fix:** Provide a visible cancel control during any upload in progress.

## Anti-pattern: Multiple file uploads with no per-file status
**What's wrong:** Uploading a batch of files shows only one aggregate progress bar with no indication of which specific files succeeded, failed, or are still in progress.
**Why it matters:** If part of the batch fails, the user can't tell which files need to be retried.
**Fix:** Show a per-file list with individual status (queued/uploading/done/failed) and a per-file retry action for any that failed.

## Anti-pattern: Downloads give no confirmation or indication of where the file went
**What's wrong:** Clicking "Download" triggers a browser download with zero in-app acknowledgment.
**Why it matters:** Especially on unfamiliar setups (managed browsers, mobile), users are sometimes unsure whether the click did anything at all.
**Fix:** Show a brief confirmation (toast: "Downloading report.pdf…") alongside the native download, especially for downloads triggered by a background/async process (e.g. "Generate report" that isn't instant) rather than an immediate direct link.

## Anti-pattern: Large exports/reports generated synchronously, blocking the UI
**What's wrong:** Clicking "Export" freezes the interface while a large file is generated server- or client-side, with the tab appearing unresponsive.
**Why it matters:** Compounds a slow operation with the appearance of a crash.
**Fix:** For anything beyond a near-instant export, generate asynchronously (background job) and notify the user when ready (in-app notification, and email for long-running exports), letting them continue using the rest of the app in the meantime.

## Anti-pattern: File names/types not previewed before upload confirmation
**What's wrong:** After selecting files, the user sees no confirmation of exactly which files were selected before they upload — just an immediate, irreversible start.
**Why it matters:** Selecting the wrong file (easy to do in a native file picker with many similarly-named files) isn't caught until it's too late.
**Fix:** Show a review list (file name, size, and thumbnail if an image) with a way to remove any wrongly-selected file before confirming the actual upload.

## Quick Checklist
- [ ] Upload/download progress is shown granularly (percentage/progress bar), not a static button for multi-second transfers
- [ ] Drop zones give clear visual feedback on drag-over
- [ ] A clickable "browse files" alternative always accompanies drag-and-drop
- [ ] File type/size is validated client-side immediately on selection, before transfer starts
- [ ] Failed uploads offer retry without a full restart; large files use resumable/chunked upload where feasible
- [ ] In-progress uploads can be cancelled
- [ ] Batch uploads show per-file status with individual retry on failure
- [ ] Downloads (especially async-generated ones) give a visible confirmation
- [ ] Large exports/reports generate asynchronously, never blocking the UI
- [ ] Selected files are reviewable (with removal option) before the actual upload is confirmed
