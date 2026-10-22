# Provider integration proposal

Standalone ToAPIs image and first-frame video calls succeeded. Includes a runnable media bridge and provider contract map. Rust provider registration, Tauri key storage and timeline import remain unverified.

## Actual source extension points

`crates/vs-providers/src/model.rs`: extend Provider enum/ALL/as_str and model catalogue; add a dedicated transport module beside fal.rs/runware.rs/openrouter.rs. Map create/poll and output files, key storage, price units and front-end provider registry. Existing source enum has only OpenRouter/Fal/Runware; ToAPIs is not currently registered.

## Verified API contract

- Base: `https://toapis.com` (no trailing /v1); Bearer API Key.
- Same-Key `GET /v1/pricing?model=...&type=image|video`, match each preset condition before POST. Missing usable quote stops submission.
- Create: `POST /v1/images/generations` or `/v1/videos/generations`. Persist submitting state before POST and task ID immediately. No automatic POST retry.
- Video first frame: `POST /v1/uploads/images` → HTTPS URL → `image_with_roles=[{url,role:"first_frame"}]` for the inspected Seedance Mini preset.
- Poll existing task at `GET /v1/images/generations/{id}` or `/v1/videos/generations/{id}`. Treat `in_progress` as running; `completed` reads `result.data[].url`; stop on failed/cancelled/expired.
- Download media without forwarding the API Key, validate PNG/MP4 and persist locally.
- Read only settled `billing.cost_usd` and `billing.credits` as actual cost. Missing billing is unknown, not zero. Do not confuse upstream credits with host-app user credits.

## Bridge code

`provider_bridge.py` wraps the runnable price-checked client; `from_file` normalizes actual downloaded files into the Wallas media shape, `save_claudia_image` preserves file output and `editor_image_fragment` returns only a media fragment. These do not register the host provider or fabricate host credit/balance fields. A downloaded-media boundary replay is not native UI acceptance.

## Minimum native acceptance

Maintainer approves integration point and target version → configure a private Key → GET pricing without paid ping → confirm one small request → preserve task ID across restart → poll once to completion → import real local output into the original tool → reconcile final billing. Keep existing suppliers, API ownership and referral links unchanged. Reference editing, masks, agent UI, cancellation and refunds require separate tests.

## Cooperation

ToAPIs can be an optional upstream API supplier, with model/usage-based pricing, agreed integration test credits and parameter/billing troubleshooting. No credits have been issued, no SLA or budget commitment is implied, and no sponsorship/exclusivity relationship is assumed.
