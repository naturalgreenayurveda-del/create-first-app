# Node.js Backend Blueprint

## Suggested stack

- Node.js + TypeScript
- Express or NestJS
- Firebase Admin SDK
- BullMQ or Pub/Sub for background jobs
- Zod/Joi for payload validation

## Core modules

- `auth`: Firebase token validation and user session endpoints.
- `media`: signed URL generation and upload completion hooks.
- `analysis`: AI job orchestration and status endpoints.
- `recommendations`: recommendation retrieval and feedback capture.
- `products`: product search adapters and normalization.
- `links`: affiliate click tracking and redirect helpers.

## Security

- JWT/Firebase token middleware on all protected routes.
- Request throttling and abuse protections on upload/analysis endpoints.
- Input sanitization and strict schema validation.
