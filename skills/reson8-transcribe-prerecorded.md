---
name: Transcribe a prerecorded audio file
description: Authenticate and transcribe a complete audio file with Reson8, optionally with diarization and word timings.
api: openapi/reson8-openapi.yml
operations: [requestToken, transcribePrerecorded]
---

# Transcribe a prerecorded audio file with Reson8

Use this to turn a complete recording into text via the Reson8 REST API.

## Auth
- Server-to-server: send `Authorization: ApiKey <api_key>` directly — no token step.
- Client-side: call `requestToken` (`POST /v1/auth/token` with the ApiKey header) to
  get a short-lived Bearer token (`expires_in` ~600s), then send `Authorization: Bearer <token>`.

## Steps
1. POST the raw audio to `transcribePrerecorded` (`POST /v1/speech-to-text/prerecorded`)
   with `Content-Type: application/octet-stream` and the file as the binary body.
2. Set query params for the container/encoding (`encoding`, `sample_rate`, `channels`)
   and, for best quality, pin `language` (or a comma list like `nl,en`).
3. For extra detail add `include_words=true`, `include_timestamps=true`,
   `include_confidence=true`, and/or `diarize=true` (splits the response into
   per-speaker `segments`, cap speakers with `max_speakers`, 1–4).
4. Bias domain terms with `custom_model_id=<id>` (see the custom-model skill).
5. Read `text` from the JSON response (or iterate `segments[]` when diarized).

## Errors
- `400 INVALID_REQUEST` — bad parameters. `401 UNAUTHORIZED` — bad/expired credential.
- `413 PAYLOAD_TOO_LARGE` — file too big (chunk it or use the realtime stream).
- `500 INTERNAL_ERROR` — retry with backoff.
