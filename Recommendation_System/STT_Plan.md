# OpenAI Speech-to-Text Implementation Plan

Date: June 20, 2026

## Final Decision

Use only:

```text
OpenAI gpt-4o-transcribe
```

This is the better choice for your chatbot because your requirement is global users.

OpenAI is a strong fit because:

- It works well for global/general speech-to-text.
- It is simple to integrate.
- It keeps the architecture clean.
- It avoids provider-routing complexity in the first version.
- It is suitable for English and multilingual users.

## Pricing

OpenAI `gpt-4o-transcribe` pricing:

```text
$0.006 per minute
$0.36 per hour
about ₹34 per hour
```

Approximate conversion used:

```text
1 USD = about ₹94.3
```

Source:

```text
https://developers.openai.com/api/docs/pricing
```

## Recommended UX

The chatbot should not directly send the transcribed text.

Use this flow:

```text
User clicks mic
  -> browser records audio
  -> frontend sends audio to backend
  -> backend calls OpenAI gpt-4o-transcribe
  -> transcript appears in the chat input
  -> user can edit the transcript
  -> user sends the message manually
```

This is the safest UX because speech-to-text can make mistakes. Letting the user edit before sending prevents wrong messages from going to the chatbot.

## Folder Structure

Add and modify files like this:

```text
interview-prep-ai/
  api/
    config.py
    main.py
    routes/
      transcribe.py              # new backend route
    services/
      speech_to_text.py          # new OpenAI STT service

  web/
    lib/
      api.ts                     # add frontend transcribeAudio() function
    components/
      chat/
        chat-input.tsx           # add mic button and transcript insertion
        voice-recorder.tsx       # new recording component

  requirements.txt               # add python-multipart if missing
  .env                           # add OpenAI/STT env variables
```

## Files To Add

### 1. `api/services/speech_to_text.py`

Purpose:

```text
Contains the OpenAI speech-to-text call.
Keeps provider logic away from route files.
```

What to put here:

```text
- validate audio file size
- call OpenAI gpt-4o-transcribe
- return normalized transcript response
```

Suggested function:

```py
async def transcribe_with_openai(
    file_bytes: bytes,
    filename: str,
    content_type: str,
) -> dict:
    ...
```

Return format:

```json
{
  "text": "transcribed user message",
  "provider": "openai",
  "model": "gpt-4o-transcribe"
}
```

### 2. `api/routes/transcribe.py`

Purpose:

```text
Creates the backend endpoint used by the frontend mic feature.
```

Endpoint:

```text
POST /api/transcribe
```

What to put here:

```text
- accept uploaded audio file
- verify Supabase auth using existing auth dependency
- call api/services/speech_to_text.py
- return transcript JSON
```

Expected request:

```text
Authorization: Bearer <Supabase access token>
Content-Type: multipart/form-data
file=<audio.webm>
```

Suggested route shape:

```py
from fastapi import APIRouter, Depends, File, UploadFile

from api.auth_supabase import get_current_supabase_user_id
from api.services.speech_to_text import transcribe_with_openai

router = APIRouter(prefix="/api", tags=["transcribe"])


@router.post("/transcribe")
async def transcribe_audio(
    file: UploadFile = File(...),
    user_id: str = Depends(get_current_supabase_user_id),
):
    file_bytes = await file.read()
    return await transcribe_with_openai(
        file_bytes=file_bytes,
        filename=file.filename or "audio.webm",
        content_type=file.content_type or "audio/webm",
    )
```

### 3. `web/components/chat/voice-recorder.tsx`

Purpose:

```text
Handles browser microphone recording.
```

What to put here:

```text
- MediaRecorder logic
- recording state
- transcribing state
- error state
- send recorded audio blob to transcribeAudio()
- return transcript to chat-input.tsx
```

States:

```text
idle
recording
transcribing
error
```

## Files To Modify

### 1. `.env`

Add:

```env
OPENAI_API_KEY=your_openai_api_key
STT_MODEL=gpt-4o-transcribe
MAX_TRANSCRIBE_SECONDS=30
MAX_TRANSCRIBE_MB=10
```

### 2. `api/config.py`

Add:

```py
OPENAI_API_KEY = os.getenv("OPENAI_API_KEY", "")
STT_MODEL = os.getenv("STT_MODEL", "gpt-4o-transcribe")
MAX_TRANSCRIBE_SECONDS = int(os.getenv("MAX_TRANSCRIBE_SECONDS", "30"))
MAX_TRANSCRIBE_MB = int(os.getenv("MAX_TRANSCRIBE_MB", "10"))
```

### 3. `api/main.py`

Import the new route:

```py
from api.routes import transcribe
```

Mount the router:

```py
app.include_router(transcribe.router)
```

### 4. `requirements.txt`

Add if not already installed:

```text
python-multipart>=0.0.9
```

Why:

```text
FastAPI needs python-multipart to accept file uploads with UploadFile.
```

Install command:

```powershell
pip install python-multipart
```

### 5. `web/lib/api.ts`

Add a frontend API helper:

```ts
export async function transcribeAudio(
  audio: Blob,
): Promise<{ text: string; provider: string; model: string }> {
  ...
}
```

What it should do:

```text
- get Supabase access token using the existing token helper
- create FormData
- append audio blob as file
- POST to /api/transcribe
- return transcript response
```

Request shape:

```ts
const form = new FormData();
form.append("file", audio, "voice.webm");
```

### 6. `web/components/chat/chat-input.tsx`

Current responsibility:

```text
This file owns the textarea value and send button.
```

Add here:

```text
- mic button
- VoiceRecorder component
- onTranscript callback
```

When transcript comes back:

```text
setValue(transcript)
focus textarea
resize textarea height
do not auto-send
```

Suggested behavior:

```text
User records voice
  -> transcript appears in the existing chat input
  -> user edits if needed
  -> user clicks send
```

## Implementation Order

Follow this order:

```text
1. Add env variables in .env.
2. Add STT config in api/config.py.
3. Add python-multipart to requirements.txt.
4. Create api/services/speech_to_text.py.
5. Create api/routes/transcribe.py.
6. Mount transcribe router in api/main.py.
7. Add transcribeAudio() to web/lib/api.ts.
8. Create web/components/chat/voice-recorder.tsx.
9. Add mic button to web/components/chat/chat-input.tsx.
10. Test recording from /chat.
```

## Testing Plan

Backend test:

```text
Start FastAPI.
Send a small webm/wav file to POST /api/transcribe.
Confirm JSON returns text.
```

Frontend test:

```text
Open /chat.
Click mic.
Record a short sentence.
Stop recording.
Confirm transcript appears in input.
Edit transcript.
Send message.
Confirm normal chat flow still works.
```

## Final Architecture

```text
Browser MediaRecorder
  -> web/components/chat/voice-recorder.tsx
  -> web/lib/api.ts transcribeAudio()
  -> POST /api/transcribe
  -> api/routes/transcribe.py
  -> api/services/speech_to_text.py
  -> OpenAI gpt-4o-transcribe
  -> transcript returned to chat input
```

## Final Recommendation

Build only OpenAI speech-to-text for now:

```text
Provider: OpenAI
Model: gpt-4o-transcribe
UX: editable transcript before send
Backend route: /api/transcribe
Frontend location: chat-input.tsx with voice-recorder.tsx
```

