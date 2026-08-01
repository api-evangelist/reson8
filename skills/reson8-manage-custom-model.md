---
name: Create and maintain a custom model
description: Create a Reson8 custom model, add and remove boosting phrases idempotently, and page through them.
api: openapi/reson8-openapi.yml
operations: [createCustomModel, listCustomModels, getCustomModel, listPhrases, addPhrases, deletePhrases]
---

# Create and maintain a Reson8 custom model

Custom models bias transcription toward domain vocabulary (medical, legal, automotive)
via text phrases — no audio or fine-tuning required.

## Auth
Send `Authorization: ApiKey <api_key>` (server-side) or `Bearer <access_token>`.

## Steps
1. `createCustomModel` (`POST /v1/custom-model`) with a JSON body `{ name, description, phrases[] }`
   (phrases must be non-empty). Capture the returned `id`.
2. `listCustomModels` (`GET /v1/custom-model`) / `getCustomModel` (`GET /v1/custom-model/{id}`)
   to inspect models and their `phraseCount`.
3. Grow vocabulary with `addPhrases` (`POST /v1/custom-model/{id}/phrases`, body `{ phrases[] }`).
   It is idempotent: the response `{ added, skipped }` reports how many were new vs already present,
   so retries are safe.
4. Remove terms with `deletePhrases` (`POST /v1/custom-model/{id}/phrases/delete`) — also idempotent
   (`{ deleted, skipped }`).
5. Page the full phrase set with `listPhrases` (`GET /v1/custom-model/{id}/phrases?page=&size=`),
   `page` zero-based, `size` 1–10000; read `total` to know how many pages remain.
6. Use the model at transcription time by passing `custom_model_id=<id>` on any speech-to-text call.

## Errors
`400 INVALID_REQUEST` (empty/blank phrase or per-model limit), `401 UNAUTHORIZED`,
`404 NOT_FOUND` (unknown model id), `500 INTERNAL_ERROR`.
