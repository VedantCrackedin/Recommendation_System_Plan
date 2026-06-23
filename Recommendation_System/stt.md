# Final STT Recommendation: Groq Batch Speech-to-Text Integration

## Recommendation Summary

Use **Groq as the primary backend-only speech-to-text provider** with a **batch / non-streaming transcription flow**.

The recommended product behavior is:

1. User clicks the microphone button in the chat input.
2. Browser records a short audio clip using `MediaRecorder`.
3. User stops recording.
4. Frontend uploads the final audio blob to the backend.
5. Backend calls Groq STT.
6. Groq returns a final transcript.
7. Frontend inserts the transcript into the chat input.
8. User reviews or edits the transcript, then sends it through the existing chat flow.

Do **not** build streaming STT, WebSocket STT, or partial transcript handling for the first version. The current app already has SSE streaming for chat responses, but it does not need streaming for speech-to-text.

---

## Why Batch STT Is the Best Fit

Batch STT is the safest and simplest option for this project because:

- The current app has no existing STT endpoint.
- There is no existing audio streaming or WebSocket infrastructure.
- The frontend chat flow is text-first.
- The transcript should be editable before the user sends it.
- Groq calls should stay on the backend to protect the API key.
- Non-streaming STT is easier to test, debug, and roll back.

The app should treat voice input as a faster way to fill the existing chat textarea, not as a separate real-time voice assistant flow.

---

## Final Architecture

```text
Browser microphone
  -> MediaRecorder records audio
  -> Frontend uploads audio Blob using multipart/form-data
  -> FastAPI /api/transcribe endpoint
  -> Backend STT provider abstraction
  -> Groq STT provider
  -> Groq audio transcription API
  -> Normalized transcript JSON
  -> Transcript inserted into chat input
  -> Existing chat send flow remains unchanged
```

Groq should be called only from the backend.

The frontend must never receive or store `GROQ_API_KEY`.

---

## Recommended Groq Model

Use this as the default:

```env
STT_MODEL=whisper-large-v3-turbo
```

Reason:

- It is optimized for speed.
- It supports multilingual transcription.
- It is a good fit for short chat-input recordings.
- Lower latency matters more than perfect transcription for this UX.

Optional accuracy-focused override:

```env
STT_MODEL=whisper-large-v3
```

Use `whisper-large-v3` later if real testing shows that interview terms, accents, or noisy audio require higher accuracy.

---

## STT Prompt Recommendation

Yes, add a Groq STT prompt, but keep it short and optional.

The STT prompt is **not** a chatbot instruction. It is a vocabulary/context hint for transcription.

It should not say things like:

```text
You are an AI interview coach. Answer the user's question in detail.
```

That is wrong because Groq STT is only converting speech to text. It is not generating the chat answer.

A good STT prompt tells the transcription model which technical terms are likely to appear, so it can spell them correctly.

Recommended environment variable:

```env
STT_PROMPT=Technical interview preparation vocabulary. Common terms: LeetCode, DSA, dynamic programming, LLD, HLD, system design, rate limiter, consistent hashing, Redis, Kafka, Docker, Kubernetes, PostgreSQL, Supabase, React, Next.js, FastAPI, GraphQL.
```

### Why This Helps

Without a prompt, speech-to-text may mishear technical terms.

Example spoken input:

```text
Explain LLD for a Redis rate limiter.
```

Possible bad transcription:

```text
Explain elder dee for a red is rate limiter.
```

With a technical glossary prompt, Groq is more likely to return:

```text
Explain LLD for a Redis rate limiter.
```

### Prompt Rules

Keep these rules:

- Keep the prompt short.
- Use it as a glossary, not as instructions for the chatbot.
- Pass it to Groq only if `STT_PROMPT` is non-empty.
- Do not include secrets, personal data, API keys, or user-specific private info.
- Do not force English unless the product should be English-only.
- Keep `STT_LANGUAGE` blank by default so Groq can auto-detect language.

---

## Environment Variables

Add these backend-only variables:

```env
GROQ_API_KEY=
STT_PROVIDER=groq
STT_MODEL=whisper-large-v3-turbo
STT_TIMEOUT_MS=30000
STT_MAX_AUDIO_SIZE_MB=10
STT_LANGUAGE=
STT_PROMPT=Technical interview preparation vocabulary. Common terms: LeetCode, DSA, dynamic programming, LLD, HLD, system design, rate limiter, consistent hashing, Redis, Kafka, Docker, Kubernetes, PostgreSQL, Supabase, React, Next.js, FastAPI, GraphQL.
```

Optional future variables:

```env
STT_FALLBACK_PROVIDER=
STT_ALLOWED_MIME_TYPES=
```

Do **not** add:

```env
NEXT_PUBLIC_GROQ_API_KEY=
```

Groq keys must never be exposed to the frontend.

---

## Backend Files to Add

```text
api/
  routes/
    transcribe.py

  services/
    stt/
      __init__.py
      types.py
      groq_provider.py
      service.py
```

### `api/routes/transcribe.py`

Create a new authenticated FastAPI route:

```text
POST /api/transcribe
```

Responsibilities:

- Require existing Supabase Bearer token authentication.
- Accept `multipart/form-data`.
- Read uploaded audio file safely with size limits.
- Validate MIME type.
- Reject empty files.
- Call the STT service.
- Return normalized transcript JSON.

Recommended response:

```json
{
  "text": "Explain LLD for a Redis rate limiter.",
  "provider": "groq",
  "model": "whisper-large-v3-turbo",
  "language": null,
  "duration_ms": 842
}
```

### `api/services/stt/types.py`

Create shared provider types.

Recommended interface:

```python
class STTProvider(Protocol):
    async def transcribe(self, input: TranscriptionInput) -> TranscriptionResult:
        ...
```

Suggested models:

```python
@dataclass
class TranscriptionInput:
    audio_bytes: bytes
    filename: str
    content_type: str
    language: str | None = None
    prompt: str | None = None

@dataclass
class TranscriptionResult:
    text: str
    provider: str
    model: str
    language: str | None = None
    raw: dict | None = None
```

### `api/services/stt/groq_provider.py`

Implement the Groq STT provider.

Responsibilities:

- Use `httpx.AsyncClient`.
- Call Groq's OpenAI-compatible transcription endpoint.
- Send multipart fields:
  - `file`
  - `model`
  - `response_format=json`
  - `temperature=0`
  - optional `language`
  - optional `prompt`
- Normalize Groq response into `TranscriptionResult`.
- Raise clean application errors for timeout, rate limit, bad response, and empty transcript.

Endpoint:

```text
https://api.groq.com/openai/v1/audio/transcriptions
```

### `api/services/stt/service.py`

Create a provider factory.

Responsibilities:

- Read `STT_PROVIDER` from config.
- Return Groq provider when `STT_PROVIDER=groq`.
- Keep route code provider-agnostic.
- Allow future providers without changing the route.

### `api/services/stt/__init__.py`

Package marker. Can export common types if useful.

---

## Backend Files to Modify

```text
api/config.py
api/main.py
requirements.txt
api/requirements.txt
```

### `api/config.py`

Add backend STT/Groq configuration:

```python
GROQ_API_KEY: str | None = None
STT_PROVIDER: str = "groq"
STT_MODEL: str = "whisper-large-v3-turbo"
STT_TIMEOUT_MS: int = 30000
STT_MAX_AUDIO_SIZE_MB: int = 10
STT_LANGUAGE: str | None = None
STT_PROMPT: str | None = None
```

### `api/main.py`

Import and mount the new route:

```python
from api.routes import transcribe

app.include_router(transcribe.router)
```

### `requirements.txt`

Add:

```text
python-multipart>=0.0.9
```

### `api/requirements.txt`

Add:

```text
python-multipart>=0.0.9
```

FastAPI needs `python-multipart` for form/file uploads.

---

## Frontend Files to Add

```text
web/
  components/
    chat/
      voice-recorder.tsx
```

### `web/components/chat/voice-recorder.tsx`

Create a dedicated recorder component.

Responsibilities:

- Request microphone permission.
- Start `MediaRecorder`.
- Stop recording.
- Build an audio `Blob`.
- Call frontend `transcribeAudio` API helper.
- Return final transcript to `chat-input.tsx`.
- Show states:
  - idle
  - recording
  - transcribing
  - error
- Handle permission denial.
- Handle unsupported browser recorder cases.
- Do not auto-send the message.

---

## Frontend Files to Modify

```text
web/lib/api.ts
web/components/chat/chat-input.tsx
web/next.config.ts
```

### `web/lib/api.ts`

Add:

```ts
transcribeAudio(audio: Blob): Promise<TranscriptionResponse>
```

Important:

- Use `FormData`.
- Include the existing Supabase Bearer token.
- Do **not** manually set `Content-Type`.
- Let the browser set the multipart boundary.
- Do not reuse a helper that forces `Content-Type: application/json`.

Suggested response type:

```ts
export interface TranscriptionResponse {
  text: string
  provider: string
  model: string
  language?: string | null
  duration_ms?: number
}
```

### `web/components/chat/chat-input.tsx`

Modify chat input behavior:

- Add a mic button.
- Render/use `VoiceRecorder`.
- Insert returned transcript into the textarea.
- Focus the textarea after transcription.
- Resize the textarea after text insertion.
- Keep existing send behavior unchanged.
- Do not auto-send transcribed text.

Recommended UX:

- If textarea is empty, set it to the transcript.
- If textarea already has text, append transcript with a space or newline.
- Let the user review before sending.

### `web/next.config.ts`

Change the microphone permissions policy.

Current behavior blocks microphone access:

```text
microphone=()
```

Change to:

```text
microphone=(self)
```

Without this change, browser microphone access may fail even if the React code is correct.

---

## Final Folder Shape

### Backend

```text
api/
  routes/
    chat.py
    transcribe.py              # new

  services/
    stt/                       # new
      __init__.py
      types.py
      groq_provider.py
      service.py
```

### Frontend

```text
web/
  lib/
    api.ts                     # modified

  components/
    chat/
      chat-input.tsx           # modified
      voice-recorder.tsx       # new
```

### Tests

```text
tests/
  test_transcribe_route.py     # new
  test_stt_groq_provider.py    # new
```

---

## API Design

### Request

```text
POST /api/transcribe
Authorization: Bearer <supabase_jwt>
Content-Type: multipart/form-data
```

Form fields:

```text
file=<audio file>
```

### Response

```json
{
  "text": "Explain LLD for a Redis rate limiter.",
  "provider": "groq",
  "model": "whisper-large-v3-turbo",
  "language": null,
  "duration_ms": 842
}
```

### Error Responses

Use consistent HTTP status codes:

```text
400  Bad request, empty file, unsupported MIME type
401  Missing or invalid auth token
413  Audio file too large
502  Groq transcription failed
503  STT provider unavailable or not configured
504  STT timeout
```

---

## MIME Type Handling

Allow common browser and audio upload types:

```text
audio/webm
audio/wav
audio/mpeg
audio/mp4
audio/ogg
audio/flac
audio/x-m4a
audio/m4a
video/webm
```

`video/webm` may appear from browser `MediaRecorder` even when the content is audio-only.

Use a small app-level file cap first:

```env
STT_MAX_AUDIO_SIZE_MB=10
```

Reject larger files in the first version. Add chunking later only if real usage requires long recordings.

---

## Groq Request Details

Backend should call:

```text
POST https://api.groq.com/openai/v1/audio/transcriptions
```

Headers:

```text
Authorization: Bearer <GROQ_API_KEY>
```

Multipart fields:

```text
file=<uploaded audio>
model=whisper-large-v3-turbo
response_format=json
temperature=0
prompt=<STT_PROMPT if non-empty>
language=<STT_LANGUAGE if non-empty>
```

Normalize response:

```python
text = response["text"].strip()
```

Return:

```json
{
  "text": text,
  "provider": "groq",
  "model": configured_model,
  "language": configured_language_or_null
}
```

If text is empty after trimming, return a clean error instead of inserting an empty message into the UI.

---

## Testing Plan

### Backend Unit Tests

Add:

```text
tests/test_stt_groq_provider.py
```

Test cases:

- Successful Groq response.
- Prompt is included when configured.
- Prompt is omitted when empty.
- Language is included when configured.
- Timeout handling.
- Rate limit handling.
- Groq 5xx handling.
- Empty transcript handling.
- Malformed Groq response handling.

### Backend Route Tests

Add:

```text
tests/test_transcribe_route.py
```

Test cases:

- Auth required.
- Valid small audio upload succeeds.
- Unsupported MIME type returns 400.
- Empty file returns 400.
- Oversized file returns 413.
- Provider failure maps to 502/503.
- Timeout maps to 504.

### Frontend Manual QA

Test manually first because the current web app may not have a frontend test setup.

Checklist:

- Mic button appears in chat input.
- Browser asks for microphone permission.
- Permission denial shows a useful error.
- Recording starts and stops correctly.
- Transcribing state appears after stopping.
- Transcript is inserted into textarea.
- Transcript is not auto-sent.
- User can edit transcript before sending.
- Existing text chat still works.
- Large recording is rejected cleanly.
- Backend/Groq failure shows a recoverable error.

---

## Migration Plan

1. Add backend STT config.
2. Add provider abstraction.
3. Add Groq provider.
4. Add `/api/transcribe` route.
5. Test backend route with curl/Postman using a real Groq key.
6. Add frontend `transcribeAudio` helper.
7. Add `VoiceRecorder` component.
8. Add mic button to chat input.
9. Update microphone permissions policy.
10. Test locally.
11. Test in staging.
12. Deploy with `STT_PROVIDER=groq`.
13. Roll back by hiding the mic UI or disabling the transcribe route; normal text chat remains unaffected.

---

## Risks and Edge Cases

Main risks:

- Browser microphone permission denied.
- No microphone device available.
- Browser does not support `MediaRecorder`.
- Mobile Safari compatibility issues.
- Unsupported MIME type from browser recording.
- Large audio files.
- Silent or empty audio.
- Empty transcript returned.
- Noisy audio.
- Accents or multilingual speech.
- Technical terms misheard.
- Groq timeout.
- Groq rate limits.
- Groq API key missing or invalid.
- Backend memory usage from large uploads.
- Permissions-Policy blocking microphone access.

Mitigations:

- Keep max file size small.
- Keep transcript editable.
- Add concise `STT_PROMPT` glossary.
- Keep `STT_LANGUAGE` blank by default.
- Use clear UI states and recoverable errors.
- Keep existing text chat untouched.

---

## Exact File-Level Change List

| Area | File | Change Needed | Reason |
|---|---|---|---|
| Backend config | `api/config.py` | Add Groq/STT env vars | Centralized config |
| Backend route mount | `api/main.py` | Include transcribe router | Expose `/api/transcribe` |
| Backend route | `api/routes/transcribe.py` | New authenticated upload endpoint | Receives audio and returns transcript |
| STT package | `api/services/stt/__init__.py` | New package marker | Organize provider code |
| STT types | `api/services/stt/types.py` | New protocol/data models | Provider abstraction |
| Groq provider | `api/services/stt/groq_provider.py` | New Groq transcription client | Primary STT implementation |
| STT factory | `api/services/stt/service.py` | New provider selector | Keeps route provider-agnostic |
| Dependencies | `requirements.txt` | Add `python-multipart` | Multipart upload support |
| Dependencies | `api/requirements.txt` | Add `python-multipart` | API deploy dependency |
| Frontend API | `web/lib/api.ts` | Add `transcribeAudio(audio: Blob)` | Upload audio to backend |
| Recorder UI | `web/components/chat/voice-recorder.tsx` | New MediaRecorder component | Mic recording lifecycle |
| Chat input | `web/components/chat/chat-input.tsx` | Add mic button and transcript insertion | User-facing STT |
| Security headers | `web/next.config.ts` | Allow microphone with `microphone=(self)` | Current policy blocks mic |
| Backend tests | `tests/test_transcribe_route.py` | New route tests | API coverage |
| Backend tests | `tests/test_stt_groq_provider.py` | New provider tests | Groq integration coverage |

---

## Implementation Order

### Phase 1: Backend Provider Abstraction

- Add `api/services/stt/` package.
- Add `types.py`.
- Add `service.py` provider factory.

### Phase 2: Groq Provider

- Add `groq_provider.py`.
- Read model, timeout, language, and prompt from config.
- Call Groq transcription endpoint.
- Normalize response.

### Phase 3: Backend Route

- Add `api/routes/transcribe.py`.
- Validate auth, file size, MIME type, and empty audio.
- Mount route in `api/main.py`.

### Phase 4: Frontend Integration

- Add `transcribeAudio` in `web/lib/api.ts`.
- Add `voice-recorder.tsx`.
- Update `chat-input.tsx` with mic UI and transcript insertion.
- Update `next.config.ts` microphone policy.

### Phase 5: Testing

- Add backend provider tests.
- Add backend route tests.
- Perform manual frontend QA.

### Phase 6: Docs and Deployment

- Add env var documentation.
- Add staging/production config.
- Verify rollback path.

---

## Final Recommendation

Implement Groq STT as a **backend-only batch transcription service** using `whisper-large-v3-turbo` by default.

Add a short optional `STT_PROMPT` containing technical interview vocabulary so Groq can better transcribe terms like `LLD`, `HLD`, `DSA`, `Redis`, `Kafka`, `Kubernetes`, `PostgreSQL`, `Supabase`, `Next.js`, and `FastAPI`.

Keep the transcript editable before sending. Do not auto-send. Do not expose Groq secrets to the frontend. Do not build streaming STT in the first version.

This gives the project the best balance of speed, implementation simplicity, safety, and production readiness.

---

## Codex Implementation Prompt

Use this prompt when asking Codex to implement the change:

```text
Implement Groq batch speech-to-text integration for this repository.

Important context:
- Use batch/non-streaming STT only.
- Do not implement streaming STT, WebSocket STT, or partial transcripts.
- Groq is the primary STT provider.
- Groq calls must happen only on the backend.
- Do not expose GROQ_API_KEY to the frontend.
- The transcript should be inserted into the chat input for user review.
- Do not auto-send the transcript.
- Preserve the existing chat SSE flow unchanged.

Backend requirements:
1. Add STT config to api/config.py:
   - GROQ_API_KEY
   - STT_PROVIDER=groq
   - STT_MODEL=whisper-large-v3-turbo
   - STT_TIMEOUT_MS=30000
   - STT_MAX_AUDIO_SIZE_MB=10
   - STT_LANGUAGE optional
   - STT_PROMPT optional
2. Add python-multipart to requirements.txt and api/requirements.txt.
3. Create api/services/stt/ with:
   - __init__.py
   - types.py
   - groq_provider.py
   - service.py
4. Create a clean STT provider abstraction.
5. Implement Groq provider using httpx.AsyncClient.
6. Call https://api.groq.com/openai/v1/audio/transcriptions.
7. Send multipart fields: file, model, response_format=json, temperature=0, optional language, optional prompt.
8. Create POST /api/transcribe in api/routes/transcribe.py.
9. Require existing Supabase Bearer auth.
10. Validate upload size, MIME type, empty files, and provider errors.
11. Return normalized JSON: text, provider, model, language, duration_ms.
12. Mount the transcribe router in api/main.py.

Frontend requirements:
1. Add transcribeAudio(audio: Blob) to web/lib/api.ts.
2. Use FormData and do not manually set Content-Type.
3. Include the existing Supabase Bearer token.
4. Create web/components/chat/voice-recorder.tsx using MediaRecorder.
5. Update web/components/chat/chat-input.tsx to add a mic button.
6. Insert returned transcript into the textarea.
7. Keep user review/edit before send.
8. Show idle, recording, transcribing, and error states.
9. Update web/next.config.ts to allow microphone access with microphone=(self).

STT prompt guidance:
- Add STT_PROMPT support on backend only.
- Treat it as a transcription glossary, not a chatbot instruction.
- Pass it to Groq only if non-empty.
- Example:
  STT_PROMPT=Technical interview preparation vocabulary. Common terms: LeetCode, DSA, dynamic programming, LLD, HLD, system design, rate limiter, consistent hashing, Redis, Kafka, Docker, Kubernetes, PostgreSQL, Supabase, React, Next.js, FastAPI, GraphQL.

Testing:
1. Add tests/test_stt_groq_provider.py with mocked httpx responses.
2. Add tests/test_transcribe_route.py with auth override and upload tests.
3. Test success, missing auth, unsupported MIME type, empty file, oversized file, Groq timeout, Groq 429/5xx, empty transcript.
4. Run available lint, typecheck, and test commands.
5. Summarize files changed, env vars, test results, and limitations.

Make the smallest safe production-ready change set.
```

---



- Groq Speech-to-Text docs: https://console.groq.com/docs/speech-to-text
- Groq Whisper Large V3 Turbo model docs: https://console.groq.com/docs/model/whisper-large-v3-turbo
- Groq Whisper Large V3 model docs: https://console.groq.com/docs/model/whisper-large-v3
