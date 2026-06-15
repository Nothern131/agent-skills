# PROJECT_CONTEXT.md

<!--
  PROJECT CONTEXT FILE
  ====================
  This file is automatically maintained by the Model Upgrade Framework.
  Update it whenever you make changes to the codebase.
  Keep it as the single source of truth for project structure and dependencies.
-->

## Directory Structure

```
[Replace with your actual project structure]
src/
├── api/
├── models/
├── services/
└── tests/
```

## Key Entity Relationships

- [Describe how models/entities relate to each other]
- Example: User (models/user.py) ←→ Order (models/order.py): one-to-many

## Critical Configuration

- [List important configuration values]
- Example: Database: PostgreSQL, connection via SQLAlchemy ORM
- Example: Auth: JWT tokens, 15min expiry

## Recent Changes

- [YYYY-MM-DD] [What changed and why]
- Example: [2024-01-15] Added payment integration to user service

## Dependencies & Contracts

- [List external dependencies and their APIs]
- Example: user_service.py expects: auth token in header
- Example: user_service.py returns: User DTO with id, name, email, created_at

## Known Issues & Workarounds

- [Any known bugs or limitations]

## Testing Strategy

- [How to run tests, what frameworks are used]
- Example: `pytest tests/` for unit tests
- Example: `npm test` for frontend tests

## Build & Deploy

- [Build commands, deployment process]
- Example: `docker-compose up -d` to start local environment
- Example: `npm run build && npm run deploy`
