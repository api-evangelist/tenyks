---
name: Create a dataset and upload its annotations
description: Authenticate, create a computer-vision dataset from a cloud-storage image location, upload its annotations, and trigger ingestion on the Tenyks platform.
api: openapi/tenyks-openapi.json
operations: [generateAccessToken, createDataset, uploadDatasetAnnotations, ingestDatasetAnnotations]
---

# Create a dataset and upload its annotations

Use this skill to get a labelled computer-vision dataset into Tenyks so it can be explored and quality-checked.

## Prerequisites
- A Tenyks API key and secret (from the dashboard: https://docs.tenyks.ai/reference/retrieving-api-keys).
- Your images and a COCO-format `annotations.json` in cloud storage (AWS S3, GCS, or Azure), plus read credentials for that bucket.
- Base host: `https://dashboard.tenyks.ai` (Premium) or `https://sandbox.tenyks.ai` (freemium).

## Steps
1. **Get a Bearer token** — `generateAccessToken`: `POST /api/auth/apikey` with `{ "api_key", "api_secret" }`. Read `access_token` from the response and send it as `Authorization: Bearer <access_token>` on every subsequent call. The token expires in 3600 seconds — refresh by repeating this call.
2. **Create the dataset** — `createDataset`: `POST /api/workspaces/{workspace}/datasets` with a `DatasetPayload`: a unique `key` (e.g. `face_detection`), `display_name`, `task_type` (`object_detection`), and `images_location` (e.g. `{ type: aws_s3, s3_uri, credentials }`). The `key` is how you address this dataset from now on.
3. **Upload annotations** — `uploadDatasetAnnotations`: `PUT /api/workspaces/{workspace}/datasets/{dataset_key}/images/annotations` with the storage `type`, `s3_uri` to the annotations file, and `credentials`.
4. **Ingest** — `ingestDatasetAnnotations`: `PUT /api/workspaces/{workspace}/datasets/{dataset_key}/ingest` to start asynchronous processing. Ingestion runs as a background job.

## Conventions & error handling
- Resources are addressed by the client-supplied `key` (dataset_key), not a server id — re-using a key targets the same dataset (see `conventions/tenyks-conventions.yml`).
- Uploads are two-phase: upload then `/ingest`. Do not skip the ingest call.
- A `400 Bad request` means invalid input or bad credentials (see `errors/tenyks-problem-types.yml`).
- The API is alpha and may change; do not hardcode assumptions about response shapes beyond what the reference documents.
