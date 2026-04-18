# Output Template

Use this structure for the exported Markdown file. Adapt section contents based on what was actually found — skip sections that would be empty, add notes where detection was partial.

Replace all `{placeholders}` with actual values.

---

## The Template

The output file should follow this structure exactly:

### Header

```
# Project Outline: {project_name}

> Generated on {YYYY-MM-DD} | Scanned: `{path_1}`, `{path_2}`
```

### Tech Stack

A table per project (or combined if single project):

```
## Tech Stack

| Layer | Technology | Version | Purpose |
|-------|-----------|---------|---------|
| Frontend | React | 18.x | UI framework |
| State | Zustand | 4.x | Client state |
| Backend | Express | 4.x | REST API |
| Database | MongoDB | 7.x | Persistence |
```

### Directory Structure

One section per project. Show the tree with annotations for key files:

```
## Directory Structure

### Frontend — React + TypeScript (`client/`)

client/src/
├── components/        (14 files)
│   ├── ui/           (6 files) — Login, PetRoom, etc.
│   ├── three/        (4 files) — 3D components (R3F)
│   └── game/         (4 files) — BattleOverlay, Inventory
├── data/             (2 files) — Static game data
├── services/         (2 files) — API client, socket ★ API calls here
├── store/            (1 file)  — Zustand store
├── styles/           (1 file)  — Global CSS
├── types/            (2 files) — Client types
├── App.tsx                     — Router setup (entry)
└── main.tsx                    — React mount (entry)

### Backend — Express + TypeScript (`server/`)

server/src/
├── models/           (4 files) — Mongoose schemas
├── routes/           (4 files) ★ API endpoints here
├── middleware/        (1 file)  — JWT auth
├── socket/           (1 file)  — Socket.io handlers
├── room-manager/     (1 file)  — Multiplayer rooms
└── index.ts                    — Server entry (entry)
```

Use `★` to mark directories that contain API endpoints or API calls.

### File Dependency Graph

```
## File Dependency Graph

The diagram below shows how files import from each other.
Arrows point from the importing file to the imported file.

` ``mermaid
graph LR
  subgraph Frontend
    App[App.tsx]
    Main[main.tsx]
    Store[store/gameStore.ts]
    API[services/api.ts]
    Socket[services/socket.ts]
    UI_Login[ui/Login.tsx]
    UI_PetRoom[ui/PetRoom.tsx]
    Three_Pet[three/Pet.tsx]
    Game_Battle[game/BattleOverlay.tsx]

    Main --> App
    App --> UI_Login
    App --> UI_PetRoom
    UI_PetRoom --> Three_Pet
    UI_PetRoom --> Store
    Game_Battle --> Store
    Game_Battle --> API
    UI_Login --> API
    Three_Pet --> Store
  end

  subgraph Backend
    Index[index.ts]
    AuthRoute[routes/auth.ts]
    PetRoute[routes/pet.ts]
    AuthMW[middleware/auth.ts]
    PetModel[models/Pet.ts]

    Index --> AuthRoute
    Index --> PetRoute
    PetRoute --> AuthMW
    PetRoute --> PetModel
  end

  subgraph Shared
    Types[shared/types/index.ts]
    GameTypes[shared/types/game.ts]
  end

  Store -.-> Types
  PetModel -.-> Types
  Game_Battle -.-> GameTypes
` ``
```

Notes:
- Use `-->` for same-project imports
- Use `-.->` (dashed) for cross-project imports
- When grouping by directory, use the directory name as the node label with file count: `Components[components/\n14 files]`

### API Endpoints

```
## API Endpoints

### Backend (`server/`)

| # | Method | Path | File | Handler | Middleware |
|---|--------|------|------|---------|-----------|
| 1 | POST | /api/auth/register | routes/auth.ts:15 | register | — |
| 2 | POST | /api/auth/login | routes/auth.ts:42 | login | — |
| 3 | GET | /api/pet | routes/pet.ts:8 | getPet | auth |
| 4 | POST | /api/pet | routes/pet.ts:22 | createPet | auth |
| 5 | PUT | /api/pet/feed | routes/pet.ts:35 | feedPet | auth |

**Total: 5 endpoints** (2 public, 3 authenticated)
```

### API Calls

```
## API Calls (Frontend)

### Frontend (`client/`)

| # | Method | Path | File | Function |
|---|--------|------|------|----------|
| 1 | POST | /api/auth/register | services/api.ts:10 | register() |
| 2 | POST | /api/auth/login | services/api.ts:18 | login() |
| 3 | GET | /api/pet | services/api.ts:26 | getPet() |
| 4 | POST | /api/pet | services/api.ts:34 | createPet() |
| 5 | PUT | /api/pet/feed | services/api.ts:42 | feedPet() |

**Base URL:** Configured via Vite proxy (`/api` → `http://localhost:4000/api`)
**Total: 5 API calls**
```

### API Cross-Reference

```
## API Cross-Reference

The diagram shows which frontend functions call which backend endpoints.

` ``mermaid
graph LR
  subgraph Frontend
    A[api.ts\nregister]
    B[api.ts\nlogin]
    C[api.ts\ngetPet]
    D[api.ts\ncreatePet]
    E[api.ts\nfeedPet]
  end

  subgraph Backend
    F[auth.ts\nPOST /register]
    G[auth.ts\nPOST /login]
    H[pet.ts\nGET /pet]
    I[pet.ts\nPOST /pet]
    J[pet.ts\nPUT /pet/feed]
  end

  A -->|POST /api/auth/register| F
  B -->|POST /api/auth/login| G
  C -->|GET /api/pet| H
  D -->|POST /api/pet| I
  E -->|PUT /api/pet/feed| J

  style A fill:#e1f5fe
  style B fill:#e1f5fe
  style C fill:#e1f5fe
  style D fill:#e1f5fe
  style E fill:#e1f5fe
  style F fill:#e8f5e9
  style G fill:#e8f5e9
  style H fill:#e8f5e9
  style I fill:#e8f5e9
  style J fill:#e8f5e9
` ``

| Status | Count | Details |
|--------|-------|---------|
| Matched | 5 | All frontend calls have a backend endpoint |
| Unmatched frontend calls | 0 | — |
| Unused backend endpoints | 0 | — |
```

If there are unmatched calls or unused endpoints, list them:

```
### Unmatched Frontend Calls
| Method | Path | File | Notes |
|--------|------|------|-------|
| GET | https://api.external.com/data | services/weather.ts:5 | External API |

### Unused Backend Endpoints
| Method | Path | File | Notes |
|--------|------|------|-------|
| GET | /api/admin/stats | routes/admin.ts:10 | No frontend caller found |
```

### Architecture Overview

```
## Architecture Overview

` ``mermaid
graph TB
  subgraph Client["Frontend (React)"]
    UI[UI Components]
    State[Zustand Store]
    Services[API Services]
    ThreeD[3D Layer — R3F]
  end

  subgraph Server["Backend (Express)"]
    Routes[Route Handlers]
    MW[Auth Middleware]
    Models[Mongoose Models]
    WS[Socket.io]
  end

  DB[(MongoDB)]

  UI --> State
  UI --> ThreeD
  State --> Services
  Services -->|REST API| Routes
  UI -->|WebSocket| WS
  Routes --> MW
  Routes --> Models
  Models --> DB
  WS --> Models

  style UI fill:#e1f5fe
  style State fill:#e1f5fe
  style Services fill:#e1f5fe
  style ThreeD fill:#e1f5fe
  style Routes fill:#e8f5e9
  style MW fill:#fff3e0
  style Models fill:#f3e5f5
  style WS fill:#e8f5e9
  style DB fill:#f3e5f5
` ``
```

This is a high-level view. Adapt it to the actual layers found in the project. Include:
- The main data flow direction (top to bottom or left to right)
- All communication protocols (REST, WebSocket, GraphQL, gRPC)
- External services if detected (database, cache, message queue, third-party APIs)
