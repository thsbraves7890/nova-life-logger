# NOVA Life Logger Build Spec

## Goal
Create an installable Android application for intentional personal audio lifelogging. The user starts a Life Log session manually. The app continuously records speech-quality audio, rotates files into approximately 20-minute chunks, uploads completed chunks while recording continues, and deletes the local raw audio only after the backend verifies complete receipt.

## Primary device
Samsung Galaxy S24 Ultra, Android 16.

## Phase plan

### Milestone 1
- Kotlin Android project
- Jetpack Compose UI
- microphone permission flow
- microphone foreground service
- Start Life Log
- Stop Life Log
- save resulting audio file
- show saved recording in History
- persistent recording notification

### Milestone 2
- automatic 20-minute file rollover
- near-seamless chunk boundaries
- keep recording while screen is locked/backgrounded
- queue old chunk as soon as the next chunk begins

### Milestone 3
- Room entities for session/chunk/bookmark/upload attempts
- DataStore settings
- Bookmark action
- Pause
- Private Mode

### Milestone 4
- WorkManager upload queue
- Retrofit/OkHttp multipart upload
- configurable backend URL
- Wi-Fi + cellular / Wi-Fi-only option
- exponential retry
- idempotent recording UUID

### Milestone 5
- SHA-256 calculation
- backend acknowledgement verification
- automatic local deletion after verified upload
- tests proving unverified audio is never auto-deleted

### Milestone 6
- polished dark-mode-first UI
- storage monitoring
- upload queue screen
- low-storage warning/protection
- manual retry for failed uploads

### Milestone 7
- Gradle build/test pass
- installable debug APK
- Samsung Galaxy S24 Ultra test checklist

## Recording behavior
Default chunk duration: 20 minutes. Allow 5/10/20/30/60 minute options later.

When a chunk reaches its limit:
1. finalize it safely
2. begin the next chunk immediately
3. calculate/store metadata
4. enqueue the completed chunk for upload
5. continue recording independently of upload activity

Suggested filename:
`2026-09-10_08-00-00_<uuid>.m4a`

Prefer AAC/M4A when robust and suitable for transcription.

## Upload payload
Multipart/form-data containing:
- audio file
- recordingId
- sessionId
- locally generated deviceId
- startedAt
- endedAt
- durationMs
- fileSize
- sha256
- bookmark timestamps
- app version
- audio format

## Backend acknowledgement
Expected shape should support at least:
```json
{
  "success": true,
  "recordingId": "...",
  "serverFileId": "...",
  "receivedBytes": 1234567,
  "sha256": "...",
  "receivedAt": "..."
}
```

Before local deletion:
- confirm successful HTTP response
- confirm success=true
- confirm recordingId matches
- confirm byte count when returned
- confirm SHA-256 when returned

If verification fails, preserve the file and retry/verify later.

## Reliability requirements
- temporary internet loss must not stop recording
- completed chunks remain queued locally until verified upload
- app UI closure must not intentionally end an active foreground recording service
- stopping recording must not cancel pending uploads
- uploads should retry after network/server failures
- phone reboot must not silently restart microphone recording
- pending uploads may resume after reboot where Android permits
- critically low storage should result in a clear warning and safe stop rather than deletion of unsent audio

## UI
Dark-mode-first, clean, modern personal-AI aesthetic.

Home should show:
- Start/Stop Life Log
- recording state
- session timer
- current chunk timer/progress
- chunks today
- uploading count
- waiting count
- uploaded count
- local audio storage
- Pause, Bookmark, Stop controls while active

History cards should clearly distinguish uploaded audio whose local copy has intentionally been removed.

## Privacy/security
- user explicitly starts recording
- microphone-active state is visible
- foreground notification remains present as required
- Private Mode and Pause stop audio capture
- no OpenAI, Supermemory, Drive, n8n secret, or permanent backend credential in the APK or repository
- no recordings/transcripts/personal data committed to Git

## Future backend
`Android -> NOVA/n8n backend -> transcription -> structured memory extraction -> Supermemory + Google Drive -> ChatGPT/NOVA`

The Android app should not perform long-term AI memory extraction itself.

## Agent implementation instruction
Read `AGENTS.md` before editing. Work milestone-by-milestone. Do not claim functionality is complete unless built/tested. Resolve compile errors before calling a milestone done. Start with Milestone 1 only.
