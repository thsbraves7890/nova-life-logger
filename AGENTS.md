# AGENTS.md

## Purpose
NOVA Life Logger is a native Android app for intentional, user-controlled ambient audio lifelogging on a Samsung Galaxy S24 Ultra running Android 16.

## Critical invariants
1. NEVER delete a local audio file until the backend has explicitly acknowledged and verified receipt.
2. NEVER commit recordings, transcripts, secrets, API keys, webhook credentials, or personal data.
3. Recording must begin only from an explicit user action and use Android's proper microphone foreground-service model.
4. Stop and Private Mode must actually stop audio capture.
5. Do not silently restart recording after the user has stopped it or after reboot.

## Architecture
- Kotlin
- Jetpack Compose + Material 3
- Foreground Service for microphone recording
- Coroutines/Flow
- Room for sessions/chunks/upload metadata
- DataStore for settings
- WorkManager for durable upload retries
- Retrofit/OkHttp for backend transport

## Planned package structure
`com.nova.lifelogger`

- `audio/` recording engine and chunk finalization
- `data/local/` Room/DataStore
- `data/remote/` backend DTOs/API
- `domain/` models/use-cases
- `service/` foreground recording service
- `upload/` WorkManager upload queue and verification
- `notification/` persistent recording controls
- `ui/home/`
- `ui/history/`
- `ui/settings/`
- `security/`
- `util/`

## Recording rules
- Use `RECORD_AUDIO`, `FOREGROUND_SERVICE`, and `FOREGROUND_SERVICE_MICROPHONE` as required.
- Declare the recorder service with `android:foregroundServiceType="microphone"`.
- Start recording from a visible user action.
- Continue while screen is off/locked and app UI is not foregrounded, consistent with Android rules.
- Default chunk length: 20 minutes.
- Finalize old chunk, immediately start next chunk, then queue the old chunk for upload.
- Minimize chunk-boundary gaps.

## Upload state machine
Suggested states:
`RECORDING -> FINALIZING -> WAITING_FOR_UPLOAD -> UPLOADING -> UPLOAD_VERIFYING -> UPLOADED -> LOCAL_FILE_DELETED`

Retryable failures must preserve the local audio file.

## Verified deletion
Before automatic deletion verify, when available:
- HTTP success
- `success == true`
- matching recording UUID
- matching byte count
- matching SHA-256

If any check fails, DO NOT DELETE.

## Privacy
This is not covert surveillance software. Keep Android microphone indicators and an ongoing foreground notification. Provide Pause, Private Mode, Bookmark, and Stop controls.

## Testing
At minimum test:
- recorder state transitions
- upload state transitions
- retry behavior
- hash verification
- verified-delete invariant
- Room DAO behavior
- settings persistence

Do not claim device or background behavior is tested unless it was actually tested.

## Development milestones
1. Foreground recording + Start/Stop + saved audio + History
2. 20-minute chunk rotation
3. Room metadata + Bookmark + Pause/Private Mode
4. WorkManager upload queue
5. SHA-256 verification + delete-after-verified-upload
6. UI/storage/network polish
7. APK build and S24 Ultra test pass
