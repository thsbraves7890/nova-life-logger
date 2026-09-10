# NOVA Life Logger

NOVA Life Logger is an Android-first personal AI life-logging project for intentional, user-controlled ambient audio capture.

The app is designed to:

- Record audio only after the user explicitly starts a Life Log session.
- Run recording in a proper Android microphone foreground service.
- Rotate recordings into approximately 20-minute chunks.
- Upload each completed chunk while the next chunk continues recording.
- Retry failed uploads safely.
- Verify server receipt before deleting the local audio file.
- Keep recording metadata after raw audio is removed from the device.
- Support bookmarks, pause, Private Mode, history, and storage monitoring.

## Planned architecture

Android app -> NOVA/n8n backend -> transcription -> memory extraction -> Supermemory / Google Drive -> ChatGPT / NOVA

AI API keys, Supermemory credentials, private webhook secrets, recordings, transcripts, and personal data must never be committed to this public repository.

## Primary device

Samsung Galaxy S24 Ultra running Android 16.

## Development approach

The project is intentionally milestone-driven. The first milestone is a reliable foreground recorder with Start, Stop, saved audio, and History. Later milestones add chunk rotation, durable upload queues, verified deletion, and backend AI processing.

See `AGENTS.md` and `docs/BUILD_SPEC.md` before making architectural changes.
