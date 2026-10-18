# Phase 6 — Pseudo-MMO Prototype

Living document for Phase 6 of the [Wiki / Portfolio Project](wiki_project.md). Holds the decisions, architecture, and subphase plan. Tasks live on the TickTick board, not here.

---

## Goal

A playable prototype of a **40k-style pseudo-MMO**: gothic sci-fi, grimdark tone, original IP.

- **Pseudo-MMO:** a shared hub where players see each other, plus instanced zones for 1–4 players. Plays like an MMO, without a seamless world or massive servers.
- **Prototype scope:** 3 classes, one starting zone, levels 1 to a low cap.
- **Built to grow:** new zones and classes are mostly new data and scenes, not new systems.

**Why this project:** it's a portfolio piece for **backend and general software engineering roles**. A game is the vehicle; the showcase is the engineering underneath — an API with a real database, a real-time server, defined contracts between services, CI, load testing, and a measured migration from Python to Rust.

---

## Decisions made

| Decision | Choice | Why |
|---|---|---|
| Scope | Pseudo-MMO: hub + instances, 3 classes, one starting zone | A true MMO is a multi-year studio project. Hub + instances keeps the MMO feel and stays buildable solo. |
| Setting | 40k-*style*, original IP | No Games Workshop names, factions, or trademarks (Space Marines, Imperium, etc.). GW enforces its IP actively. |
| Client engine | Godot 4, GDScript | Text-based scene files Claude Code can read and edit; Python-like language; performance is fine for stylized scenes and small instances. |
| Camera | Isometric | Much cheaper than third-person in camera code, animation, and level work. Fits the server's ground-plane simulation. Tactical, grimdark look suits the setting. |
| Movement input | WASD, camera-relative | Direct, action-oriented feel. Client sends a movement direction each tick; the server moves the character and resolves collisions against the zone's walkable area. Needs client-side prediction for the player's own character, or input feels laggy. Pathfinding stays server-side for enemies only. |
| Godot editor location | Windows-native Godot opens the project inside the WSL filesystem via `\\wsl.localhost\...` (provisional) | Godot's Vulkan renderer is poorly supported under WSL (software fallback, crashes). Keeps one repo, in WSL, as the single source of truth. WSL2 forwards `localhost`, so the client reaches the API and game server without extra config. **Fallback:** if file scanning/imports get slow once real assets land (6.4), use a second clone on the Windows side for client work. |
| Meta backend | FastAPI, PostgreSQL, SQLAlchemy, Alembic, Redis, JWT | Known language (Python), matches the original Phase 6 plan, standard backend stack worth showing. |
| Real-time game server | Python (asyncio), server-authoritative; later rewritten in Rust | See note below. Gives the Python → Rust path a real, measurable target. |
| Language path | Python first → Rust via PyO3 modules → full Rust game server | Start productive; migrate where it measurably pays off; document each step with before/after numbers. |
| Rust learning | Written by hand, not delegated | If Claude writes the Rust, I don't learn Rust, and it shows in interviews. The Rust Book and Rustlings first. |
| Networking | Multiplayer from day one; server decides all game state | Retrofitting multiplayer onto a single-player game is the classic trap. Clients send input, not results. |
| Content | Data, not code | Classes, abilities, enemies, quests, and loot tables live in data files. New content shouldn't need new systems. |
| Contracts | Language-neutral: OpenAPI spec for the API, documented message protocol for the game server | Any service can be rewritten (e.g. in Rust) without the others noticing. Contract tests run against every implementation. |
| Local infra | Docker Compose | One command starts API, Postgres, Redis, and game server. |
| CI | GitHub Actions | Already familiar from Phase 3. Tests, lint, and later Rust builds. |
| Assets | Blender (+ Blender MCP), Meshy/Tripo for models, Mixamo for animations, glTF/GLB, Git LFS | Claude Code doesn't make art. Placeholders until 6.4; check licenses before use. |
| Hosting | Single cheap VPS when needed | Kubernetes/Agones only if a real limit forces it. |

**Game server note:** earlier discussion assumed a Godot headless server. Switched to a separate Python server because (a) it's a backend showcase rather than engine scripting, (b) it gives the PyO3 step (6.6) real CPU-bound Python code to optimize, and (c) a Rust rewrite (6.7) then replaces Python with Rust behind the same protocol. Cost: the server can't reuse Godot's physics, so it simulates a simplified world — positions on the ground plane, circle colliders, a navmesh exported from the zone. The client renders full 3D.

### Alternatives considered

| Option | Verdict | Reason |
|---|---|---|
| Unreal (C++) | Rejected | Best graphics, but heavy, and Blueprints/binary assets are invisible to Claude Code. |
| Unity (C#) | Rejected | Mature and fast, but editor-centric; 2023 licensing history. |
| Bevy (Rust) | Rejected for now | Very fast and all code, but three learning curves at once (Rust + game dev + netcode), frequent breaking releases, no editor. Rust comes in through the backend instead. |
| Godot headless game server | Replaced | See note above. |
| Nakama (ready-made game backend) | Rejected | Saves time but removes the backend work this project exists to show. |
| Full Rust from day one | Rejected | Python first gives a working baseline, faster progress, and real numbers to compare against. |

---

## Guiding principles

- **Practices are answers to problems** (from the main project). Start deliberately simple — JSON over WebSockets, one process — and change things only when measurement shows a reason. Each change is an article.
- **One checkpoint per subphase.** No new system until the current checkpoint visibly works.
- **Boundaries first.** Contracts are written before the code on either side of them.
- **Delegate the mechanical, own the core.** Claude Code: boilerplate, config, UI wiring, test scaffolding. Me: protocol design, all Rust, all articles. Log what delegation cost and saved as `delegated` progress-log rows — that's Phase 7 material.

---

## Architecture

```
┌──────────────────────┐   HTTPS (REST, JWT)   ┌───────────────────────┐
│  Godot 4 client      │──────────────────────▶│  Meta backend         │
│  GDScript, 3D render │                       │  FastAPI              │
│                      │                       │  accounts, characters,│
│                      │                       │  inventory, matchmaking│
└──────────┬───────────┘                       └──────┬─────────┬──────┘
           │ WebSocket (game protocol)                │         │
           ▼                                          ▼         ▼
┌──────────────────────┐   service auth (REST)   ┌──────────┐ ┌───────┐
│  Game server         │────────────────────────▶│ Postgres │ │ Redis │
│  Python asyncio      │                         └──────────┘ └───────┘
│  → Rust (Tokio) 6.7  │◀──── instance registry, chat pub/sub ───┘
│  hub + instances     │
└──────────────────────┘
```

- **Meta backend:** everything persistent and not real-time. Mostly waiting on the database, so Python is fine here long-term.
- **Game server:** fixed tick loop (start at 20 Hz), authoritative movement and combat, one process can host several instances. Verifies the player's JWT on connect; writes progress (XP, loot) back through the meta backend.
- **Client:** sends input, renders state, interpolates other players. Never decides outcomes.

**Proposed repo layout** (confirm at 6.0), following the "one repo per project" decision:

```
game/
├── client/       # Godot project
├── api/          # FastAPI meta backend
├── server/       # Python game server (later: server-rs/)
├── contracts/    # OpenAPI spec, game protocol spec, shared test fixtures
├── content/      # classes, abilities, enemies, quests, loot (data files)
├── docker-compose.yml
└── .github/workflows/
```

---

## Subphases

Rough effort: 6.0–6.5 around **3–5 months part-time**. 6.6–6.7 depend on how Rust learning goes. Estimates, not commitments.

### 6.0 — Project setup
- Decide: setting and names — game name (also the repo name), factions, tone document.
- Decide: confirm the repo layout above.
- Create the repo with that layout; Git LFS for assets from the first commit.
- Docker Compose with API, Postgres, and Redis (empty services).
- CI skeleton: lint (Ruff), tests, runs on PRs; same ruleset as `devlog` (require PR, require checks).
- Dependabot for the `github-actions` ecosystem (lesson from the Node 20 deprecation).
- `CLAUDE.md` with project conventions and what not to delegate.
- Install Godot on Windows; open `client/` via `\\wsl.localhost\...` and confirm the editor and a test scene run.
- Confirm the Windows client reaches services in WSL on `localhost`.

**Checkpoint:** `docker compose up` starts all services; a PR runs CI green.

### 6.1 — Meta backend
The original Phase 6 project.
- Decide: the three class archetypes (e.g. melee, ranged, support) and their names — character creation needs them.
- FastAPI app; SQLAlchemy models; Alembic migrations.
- Accounts: register, login, password hashing (argon2), JWT issue/refresh.
- Characters: create (pick one of 3 classes), list, delete.
- OpenAPI spec committed to `contracts/` and checked in CI.
- Tests against a real Postgres in CI (service container), not mocks.

**Checkpoint:** create an account, log in, create a character — all via the API, all covered by tests, CI green.
**Article:** designing the meta backend; why tests hit a real database.

### 6.2 — Networked skeleton
- Game protocol v0 written in `contracts/`: message types, fields, versioning. JSON over WebSockets — deliberately simple.
- Python game server: asyncio, fixed tick, verifies JWT, authoritative movement from WASD direction input (sequence-numbered), with speed checks.
- Client-side prediction for the player's own character, with reconciliation when the server's position disagrees.
- Godot client: login screen calling the API, character select, a flat test scene, capsule players.
- Client-side interpolation for other players.

**Checkpoint:** two clients on one machine log in and see each other move smoothly; a client sending impossible movement gets corrected by the server.
**Article:** server-authoritative movement, prediction and reconciliation, and what "the server decides" means in practice.

### 6.3 — Combat core
- Abilities, cooldowns, and resources defined in `content/` data files.
- One class end to end first, then the other two.
- One enemy type with server-side AI: aggro, chase, attack, leash.
- Damage, death, respawn, XP gain.
- Placeholder art is fine.

**Checkpoint:** two players kill an enemy together; all three classes playable.
**Article:** content as data — how abilities are defined and validated.

### 6.4 — Starting zone
- Instance system: one instance per party, created and torn down on demand.
- Zone layout, enemy spawns, navmesh exported for the server.
- Decide: level cap for the starting zone, 5 or 10.
- 3–5 quests and loot tables as data.
- Inventory and progress persisted via the meta backend (service-to-service auth).
- Real assets: style guide first, then the pipeline (concept images → image-to-3D → Blender cleanup → Mixamo → glTF). License check for every source.

**Checkpoint:** a fresh character levels from 1 to the cap in the zone with a friend; progress survives logout.
**Article:** the AI-assisted asset pipeline — what worked and what still needed hand work.

### 6.5 — Hub
- Shared hub instance with more players than a zone instance.
- Instance registry in Redis; party formation; "enter zone" and "return to hub" flow.
- Chat via Redis pub/sub.
- Bot clients for load testing: headless scripts speaking the protocol.

**Checkpoint:** players meet in the hub, form a party, enter an instance together, and return. A load test produces **baseline numbers** (tick time, CPU, players per process) for 6.6.
**Article:** where the Python server starts to hurt, with numbers.

### 6.6 — First Rust module (PyO3)
- Profile the server under bot load (`py-spy`) and pick the hottest CPU-bound piece — likely spatial queries, enemy AI, or pathfinding.
- Rewrite that piece in Rust with PyO3 + maturin; Python imports it like any package.
- CI builds the Rust wheel.
- Same load test, compared against the 6.5 baseline.

**Checkpoint:** measured improvement on the same load test; both versions covered by the same tests.
**Article:** first Rust in production code — before/after numbers and what was hard.

### 6.7 — Rust game server
- Rewrite the game server in Rust (Tokio) against the same protocol.
- Contract tests from `contracts/` run against both the Python and Rust servers.
- Revisit transport and encoding (UDP, binary format) only if measurements say so.

**Checkpoint:** the unchanged Godot client plays against the Rust server; load test comparison against both Python versions.
**Article:** rewriting a service behind a stable contract.

### Beyond the prototype
More zones and classes; deploy to a VPS; possibly the meta backend in Rust (Axum) — for learning, stated honestly, since it's I/O-bound.

---

## Learning track (runs in parallel)

| When | What | How |
|---|---|---|
| 6.0–6.2 | Godot basics | Official docs; the "your first 3D game" tutorial. |
| 6.1–6.5 | Rust fundamentals | The Rust Book + Rustlings, by hand. Ownership will feel like a stricter RAII (C++ background helps). |
| 6.4–6.5 | Small Rust exercises | A standalone tool or library, written without delegation, before touching the game. |
| 6.6 onward | Rust in the project | Claude Code as a reviewer and helper once I can read and judge its Rust. |

---

## Risks

| Risk | Mitigation |
|---|---|
| Scope creep — MMOs grow without limit | Pseudo-MMO scope; checkpoints; no new system before the current checkpoint works. |
| Netcode bugs only show up in play | Bot clients and two-client tests from 6.2; log every server tick decision in debug mode. |
| Rust learning curve stalls progress | Python baseline works first; Rust enters in small steps (6.6 before 6.7). |
| Godot editor and WSL don't mix well | Editor runs on Windows, files stay in WSL. Watch import speed at 6.4; fall back to a Windows-side clone if needed. |
| Art style inconsistency; asset licensing | Placeholders until 6.4; consistent concept images; log the license of every asset. |
| Games Workshop IP | Original names, factions, and designs only. |
| Python server performance | Intended. It's the measured reason for 6.6 and 6.7, not a failure. |

---

## Progress log

Mode: `coached`, `delegated`, `solo`, or `—` (before modes were tracked). See [`wiki_project.md`](wiki_project.md#progress-log).

| Date | Mode | What happened |
|---|---|---|
| 2026-10-12 | — | Idea: 40k-style MMORPG prototype. Scoped down to a pseudo-MMO (hub + instances). Reviewed AI 3D tooling: Blender MCP for scene work, Meshy/Tripo for generated assets. |
| 2026-10-13 | — | Engines compared (Godot, Unity, Unreal, Bevy). Portfolio target set: backend / general SE. Language path decided: Python first, then Rust. Game server moved from Godot headless to a separate Python service. Subphases 6.0–6.7 drafted. |
| 2026-10-13 | — | Camera: isometric. Godot editor: Windows-native, opening the repo inside WSL (fallback: Windows-side clone if imports get slow). Movement: WASD, camera-relative; client-side prediction added to 6.2. |
