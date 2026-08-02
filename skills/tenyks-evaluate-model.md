---
name: Register a model and upload its predictions for evaluation
description: Authenticate, register a model against an existing Tenyks dataset, upload its predictions from cloud storage, and trigger ingestion so Tenyks can evaluate model performance.
api: openapi/tenyks-openapi.json
operations: [generateAccessToken, createModel, uploadModelPredictions, ingestModelPredictions]
---

# Register a model and upload its predictions for evaluation

Use this skill to get a model's predictions into Tenyks so its performance can be compared against ground-truth annotations. This assumes the dataset already exists (see the "Create a dataset" skill).

## Prerequisites
- A Tenyks API key and secret.
- An existing dataset `key` in your workspace.
- A predictions file in cloud storage (AWS S3, GCS, or Azure) plus read credentials.

## Steps
1. **Get a Bearer token** — `generateAccessToken`: `POST /api/auth/apikey` with `{ "api_key", "api_secret" }`; use the returned `access_token` as `Authorization: Bearer <access_token>`.
2. **Create the model** — `createModel`: `POST /api/workspaces/{workspace}/datasets/{dataset_key}/model_inferences` with a `ModelPayload`: `display_name`, unique `key` (e.g. `yolo_v8`), `iou_threshold` (default `0.5`), `confidence_threshold` (default `0.5`).
3. **Upload predictions** — `uploadModelPredictions`: `PUT /api/workspaces/{workspace}/datasets/{dataset_key}/model_inferences/{model_key}/predictions` with the storage `type`, `s3_uri` to the predictions file, and `credentials`.
4. **Ingest** — `ingestModelPredictions`: `PUT /api/workspaces/{workspace}/datasets/{dataset_key}/model_inferences/{model_key}/ingest` to start asynchronous processing.

## Conventions & error handling
- Models are addressed by `model_key` under their parent `dataset_key`.
- `iou_threshold` and `confidence_threshold` are required and are sent as strings.
- Upload then `/ingest` — the two-phase pattern applies to predictions too.
- Handle `400 Bad request` for invalid payloads or credentials (`errors/tenyks-problem-types.yml`).
- Token TTL is 3600s; refresh before long ingestion waits.
