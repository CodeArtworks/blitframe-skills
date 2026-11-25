---
name: blitframe-render
description: Renders video on Blitframe and delivers the results. Covers MP4, HLS and frame renders, quality, what a render costs in render seconds and when it is refunded, plan limits, waiting for a result, previewing with frames, batch renders, cancelling, ZIP archives of many videos, share links that play in the browser, and HTML5 ad packages for ad networks. Use when the user asks to render, export, download, preview, share or deliver a video, animation or still frames, to estimate or explain a cost or a refused or failed render, or to package a video as an HTML5 or playable ad, whether or not they name Blitframe.
license: MIT
---

# Render and deliver

A render turns one version of a project into an MP4, an HLS stream or still frames. MCP tools are written `blitframe:<tool>`. Read the `blitframe` skill first.

## Before you render

1. The version's `renderable` is `true` (`blitframe:get_project` shows it for `main`'s head; for another variation, `blitframe:get_version` with its `headVersionId` from `blitframe:list_variations` with its name as `q`). A version with `problems` is refused (`422 version_not_renderable`), and so is an empty project.
2. `blitframe:get_usage` (`GET /usage`) shows `renderSecondsRemaining` and `limits`:
   - `maxDurationSeconds` caps an MP4 or HLS video; `maxLongEdge` and `maxShortEdge` cap the composition's size in every format. `null` means Blitframe's own maximum.
   - `watermark: true` means outputs carry a small "Made with Blitframe" mark.
   - `noTrial: true` or `planEnded: true` means the workspace cannot render until a plan starts. Tell the user; do not try.
3. Preview before you spend: a few frames cost far less than the video.

## Queue a render

`blitframe:render` (`POST /renders`):

```json
{ "projectId": "<project>", "format": "mp4", "quality": "high", "name": "Summer teaser" }
```

| Field | Values |
|---|---|
| `projectId` or `versionId` | Exactly one. `projectId` renders the project's head at that moment; `versionId` renders that exact version |
| `variation` | With `projectId` only: render the head of this variation instead of `main`'s (the `blitframe-variations` skill) |
| `format` | `mp4` (default), `hls` or `frames` |
| `codec` | `mp4` and `hls` only: `h264` (default), `hevc` or `av1`; `hevc` and `av1` need a plan with advanced encoding. HEVC and AV1 are smaller at the same quality but play on fewer devices; HTML ads need `h264` |
| `quality` | `low`, `medium`, `high` (default) or `extraHigh`. It sets the bitrate for the composition's size and frame rate: `low` is half of `high`, `medium` three quarters, `extraHigh` one and a half |
| `frames` | Only with `frames`, and required there: `{ "start": 0, "end": 30, "format": "png" }`. `end` is exclusive, so `start: 60, end: 61` is one frame. At most 1,000 frames. `png` (default) and `tga` keep transparency; `jpeg` does not |
| `audio` | `{ "path": "audio/music.mp3", "offsetSeconds": 0 }`: audio for this render only, overriding the version's soundtrack. Frame renders ignore it |
| `name` | The MP4's download file name, up to 200 characters |
| `replacements` | New text, fonts, images or video by layer for this render alone; the head does not move (the `blitframe-variations` skill) |

Size and frame rate come from `data.json`; a render request cannot change them.

The answer's `sourceVersionId` is the version the request started from (the head, or the `versionId` sent), and `versionId` is the exact version rendered. They are the same unless the render has `replacements`: then Blitframe makes the render its own version, and `versionId` is `null` until it is ready, within seconds.

Send an idempotency key with every render, so a retry after a timeout or a lost answer cannot charge twice: `idempotencyKey` over MCP, the `Idempotency-Key` header over REST (1 to 200 characters, no `/`, not starting with `share:`), a new value per render. The same request with the same key returns the same render for 24 hours (over REST with `Idempotent-Replayed: true`), without a second charge, even when the head has moved since. The same key with a different request fails with `idempotency_key_reused`: use a new key. A render you deleted frees its key.

## Wait, then hand over

`blitframe:get_render` with `waitSeconds` up to 50 (`GET /renders/{renderId}` with `Prefer: wait=50`) holds the answer until the render is `succeeded`, `failed` or `cancelled`, or the time passes. Call it again while `status` is `queued` or `rendering`.

- Look at the result before you report it. Over MCP a succeeded render comes back with images: up to 8 of its first frames, or a video's poster (`includeImages: false` leaves them out). Over REST, download `output.frameUrls[0]` of a frame render, or `output.posterUrl` of a video, and look at that file.
- `output.mp4Url` downloads the video. An `hls` render also has `output.hlsUrl`, a playlist for HLS players. A frame render has `output.frameUrls`, in order (only from this call, not from lists).
- `output.posterUrl` is a JPEG still from the middle of a video.
- Links last an hour (`output.expiresAt`). Get the render again for fresh ones.
- An `hls` render can be watched while it renders: once `playback` is `streaming`, `output.hlsUrl` already plays.

## What it costs

Render seconds = the video's length in seconds, rounded up, at least 1, at any resolution. A frame render costs frames ÷ 30, rounded up, at least 1.

Length is `(op - ip) / fr`. A 15-second video costs 15 render seconds, in 720p, 1080p or 4K alike.

The charge is taken when the render is queued. A render that fails or is cancelled is refunded in full, except:

1. an `hls` render that had already started streaming,
2. a render that ended `render_time_limit`: its frames took longer than the render seconds it costs, and
3. a render queued before the balance was reset (a new billing period, an ended trial or plan, or a payment left unpaid for 3 days).

None of these applies when the failure was on Blitframe's side (`interrupted`, `internal_error`, `stopped_by_support`), but after a reset that refund fills the balance only up to the new period's allowance. Every render shows `chargedSeconds` and `refundedSeconds`, the seconds that actually came back.

## When a render is refused or fails

Refused when queued, with nothing charged:

| Answer | Meaning | What to do |
|---|---|---|
| `422 version_not_renderable` | The version has problems, its files total more than 8 GB, a video was asked of an odd width or height, or what the render writes alone needs more than 40 GB | Fix the version or render frames for an odd size. Over 40 GB: frames, render fewer; video, choose a lower `quality` or a shorter video |
| `403 plan_limit` | Longer or larger than the plan allows | Make the composition shorter or smaller in `data.json`, or tell the user what the message says lifts the limit |
| `402 insufficient_render_seconds` | The balance is too low, there is no trial, the trial or plan ended, or a failed payment is still unpaid 3 days after the period ended | Tell the user what the message says. Do not retry |
| `402 payment_disputed` | A payment is disputed with the card's bank | Tell the user. Renders are paused until it is closed |

Failed after it started (`status: "failed"`, with `error.code`):

| `error.code` | What to do |
|---|---|
| `render_failed` | The engine could not render the version. Check the composition against the supported features (the `blitframe-data-json` skill) |
| `render_time_limit` | Its frames took longer than the render seconds it costs (at least 5 seconds), and it is not refunded. Simplify the composition (fewer or lighter 3D models, effects, precompositions or video layers) before rendering again |
| `render_timed_out` | It ran past its time limit (30 minutes for copying its files, plus its start-up and frame time), or did not reach its first frame within 1 minute (refunded: render again). Otherwise make the composition shorter or simpler. When the message says the videos and images it converted are kept, render again: it goes on from them. When it names a file or a step it could not finish in time, rendering again stops there too: do what it says, such as trimming that video or uploading it as an 8-bit H.264 MP4 of at most 4,096 pixels |
| `version_not_renderable` | With `replacements` and `versionId` still `null`: the replacements' files would take the render's version over 8 GB, so none was made. Use smaller replacement files. With `replacements` otherwise: the version made for the render may have problems; `blitframe:get_version` with its `versionId` lists them. With no problems, it needs more than 40 GB on the render server: declare images at their file's size, use fewer or shorter transparent or odd-sized videos, or render fewer frames or a shorter video. Rendering the same version again fails the same way |
| `interrupted` | Render again. When the message names a missing file, add that file again with an edit and render the new head |
| `internal_error` | Render again. If it repeats, `blitframe:report_problem` (`POST /problem-reports`) with the `renderId` |
| `stopped_by_support` | Read the message |
| any other code | The version failed a check when the render started, or the render's `replacements` could not be applied (`versionId` is still `null`; for example an upload was already used). Fix what the message names, in a new version or in the replacements, and render again |

A render is never retried for you. Rendering again is a new render. Each upload works once, so to render the same `replacements` again, render the first render's `versionId` (when set) without replacements, or send them with new uploads.

## Many renders

- **Queue up to 1,000 at once:** `blitframe:create_render_batch` (`POST /projects/{projectId}/renders/batch`) with `renders: [{ "format": ..., "name": ..., "replacements": ... }]`. Every render starts from one version of the project: `versionId`, or when it is left out the head of `main`, or of `variation`; the answer's `versionId` names it. Send an idempotency key with the batch too. Each render has its own `format`, `quality`, `frames`, `audio`, `name` and `replacements` (one video per row of data: the `blitframe-variations` skill). All or nothing: if one item is refused, nothing is queued or charged, and the message starts with the item (`renders[1]: ...`).
- **Check up to 1,000:** `blitframe:get_render_statuses` (`POST /renders/statuses`) with `renderIds`. Ids that are not the workspace's come back in `notFound`.
- **Deliver as one ZIP:** `blitframe:create_download_archive` (`POST /renders/download-archive`) with `renderIds` (succeeded MP4 or HLS renders, 8 GB in total) and an optional `name`; then `blitframe:get_download_archive` with `waitSeconds` (`GET /renders/download-archive/{archiveId}` with `Prefer: wait=50`) until `status` is `ready`, and give the user its `url`. Archives are kept 24 hours.
- Queued renders all run; a busy queue gives each workspace a share of capacity by plan, and plans with a priority render queue start first. A workspace on the free trial renders one video at a time, so the rest of a trial batch stays `queued` until the one before it finishes: keep waiting, it is not stuck.
- `codec` `hevc` or `av1`, and `encoding` (bitrate, keyframe interval, H.264 profile), need a plan with advanced encoding: `403 plan_feature` otherwise. A full workspace gets `403 storage_limit`: delete old renders or projects.

## Cancel and delete

Ask the user first.

- `blitframe:cancel_render` (`POST /renders/{renderId}/cancel`) stops a queued or running render and refunds it, with the exceptions above. A running render comes back still `rendering` and turns `cancelled` within seconds. A finished one answers `render_finished`.
- `blitframe:cancel_renders` (`POST /renders/cancel`) cancels up to 1,000.
- `blitframe:delete_render` (`DELETE /renders/{renderId}`) deletes a finished render and its files. A queued or running one answers `render_active`: cancel it first. Deleting does not change the balance.

## Share links

A share link is a public page that plays a project's video, without an account: `blitframe:create_share_link` (`POST /projects/{projectId}/share-links`) with an optional `quality`, and `versionId` or `variation`. Give the user the answer's `url`.

- **Ask the user before creating one.** Anyone with the link can watch.
- 3D models need a plan that includes them (`limits.threeD` in `blitframe:get_usage`): a version holding a `.glb` or `.gltf` file renders only there, otherwise `403 plan_feature`.
- Share links need a plan that includes them (`limits.shareLinks` in `blitframe:get_usage`); otherwise `403 plan_feature`, and a workspace's links stop playing while its plan lacks them.
- Without `versionId` the link follows the head of `main`, or of the `variation` you name, and is deleted with that variation; with `versionId`, the link stays on that version.
- Opening the page renders nothing. If no plain render of that version and quality exists yet, the first visitor to press Play queues one HLS render, charged to the workspace. To avoid that, render the version yourself first at the link's quality with the default `h264` codec, without `name`, `audio`, `replacements` or `encoding`.
- A project can have 50 links. `blitframe:list_share_links` (`GET /projects/{projectId}/share-links`) lists them; `blitframe:delete_share_link` (`DELETE /share-links/{linkId}`) revokes one.

## HTML5 ads

`blitframe:export_html_ad` (`GET /renders/{renderId}/html?platform=...&clickUrl=...`) wraps a succeeded MP4 or HLS render as an HTML5 ad and returns a download `url` (one `index.html` or an `ad.zip`, valid for an hour).

| `platform` | Notes |
|---|---|
| `generic` (default) | One HTML file with the video embedded |
| `googleCampaignManager360`, `googleDisplayVideo360` | ZIP for rich media |
| `theTradeDesk` | ZIP with a backup image. The video must be under 1.1 MB, or at most 2.2 MB with `interaction: "tapToContinue"` |
| `adform` | ZIP with a manifest. Requires `clickUrl` |
| `googleAppCampaigns`, `meta`, `tikTok`, `pangle` | Playable ad as a ZIP |
| `appLovin`, `unityAds` | Playable ad as one HTML file. Requires `clickUrl` |
| `mintegral`, `moloco`, `liftoff` | Playable ad as one HTML file |

- `clickUrl` is an absolute `https://` URL. `interaction` is `video` or `tapToContinue`; playable platforms accept only `tapToContinue`.
- The whole ad must be under 5 MB (`too_large`): render at a lower `quality`, or shorten the video.
- `render_not_ready` means the render is not finished (wait for it), or it failed or was cancelled (render the version again); `backup_unavailable` means retry.

More: `https://codeartworks.com/docs/blitframe/renders.md` and `https://codeartworks.com/docs/blitframe/sharing.md`.
