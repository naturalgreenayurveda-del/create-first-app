# Development Roadmap (Detailed)

## Milestone 1 — Project Foundation

- Initialize mono-repo modules (`mobile_app`, `backend`, `ai-services`, `shared`).
- Configure linting, formatting, commit hooks, and CI pipelines.
- Provision Firebase (Auth, Firestore, Storage) and environment config strategy.
- Build auth and token verification end-to-end.

**Exit criteria**
- User can sign up/log in and access authenticated API routes.

## Milestone 2 — Upload and Analysis Orchestration

- Implement photo/video upload from Flutter.
- Generate signed upload URLs from backend.
- Store media metadata and create analysis sessions.
- Add queue-based async processing triggers for Python AI service.

**Exit criteria**
- Uploaded media starts an analysis session and exposes status polling.

## Milestone 3 — AI Attribute Detection MVP

- Build preprocessing pipeline (frame extraction and quality gates).
- Implement MediaPipe body and face landmark extraction.
- Implement OpenCV skin tone estimation.
- Return normalized attributes to Node backend.

**Exit criteria**
- System outputs body shape, face shape, and skin tone with confidence values.

## Milestone 4 — Recommendation Engine v1

- Build rule engine for outfit, hairstyle, accessories, and shoes.
- Add budget/style constraints in candidate generation.
- Persist recommendation bundles and expose via API.

**Exit criteria**
- App displays recommendation categories with explanation bullets.

## Milestone 5 — Product Search and Buy Links

- Integrate one or more commerce/search providers.
- Normalize products and map recommendations to purchasable items.
- Implement affiliate link generation and click tracking.

**Exit criteria**
- User can open valid buy links from recommendation cards.

## Milestone 6 — UX Polish and Personalization

- Add history, saved looks, and feedback controls.
- Introduce ranking model using user interaction signals.
- Improve latency, caching, and retry handling.

**Exit criteria**
- Recommendation relevance and conversion metrics improve over MVP baseline.

## Milestone 7 — Production Launch

- Security hardening and privacy compliance checks.
- Monitoring, alerting, SLOs, and rollback strategy.
- Closed beta, bug triage, and public launch.

**Exit criteria**
- Stable release candidate with observability and incident response readiness.
