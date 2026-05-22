# Project Spec Index

This repository is a single Go + React application. Use the layer that matches
the files you are changing, and read the shared thinking guides for cross-layer
work.

| Layer | Scope |
| --- | --- |
| [Backend](./backend/index.md) | Go server, Gin routes, controllers, services, GORM models, relay adapters, billing, settings, middleware |
| [Frontend](./frontend/index.md) | React UI under `web/default`, legacy React UI under `web/classic`, API clients, routes, stores, components |
| [Thinking Guides](./guides/index.md) | Cross-layer and code-reuse checks to run before non-trivial changes |

For work that touches both Go and React, read:

- backend directory, error, and database rules for API contracts and persistence
- frontend data-flow/routing rules for how responses are consumed
- `guides/cross-layer-thinking-guide.md` before changing request or response
  formats
