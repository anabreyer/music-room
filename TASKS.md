# Music Room — Project TODO List

---


## Legend

- **jischoi** — Backend: Authentication / Security / Music Playlist Editor / CI
- **aaduan-b** — Backend: Data Modeling / Music Track Vote / IoT / Offline
- **aseisenb** — Frontend: Mobile UI / Web Responsive
- **Shared** — Architecture decisions, load testing
- **Mandatory** | **Bonus**
- Status: `[ ]` Not started / `[~]` In progress / `[x]` Done

---

## Summary

| | Count | Done | In Progress |
|---|---|---|---|
| Mandatory | 47 | 0 | 0 |
| Bonus | 15 | 0 | 0 |
| **Total** | **62** | **0** | **0** |

---

## Project Requirements Checklist

> Derived directly from the subject. Each owner is responsible for writing and maintaining tests for their own items whenever a feature is added or changed.

### V.1 — User

> Note 1: User model must be agreed upon by jischoi and aaduan-b before individual work starts.
> Note 2: Friend relationship API (aaduan-b) must be completed before jischoi can implement friends-only access control.
> Note 3: aseisenb should use mock data for UI until backend APIs are ready.

- [~] [Shared] User model definition (auth fields + profile fields) — prior to all other work · spec: `docs/design/user-model.md` (awaiting jischoi sign-off)
- [ ] [aaduan-b] Profile data structure: public / friends-only / private fields
- [ ] [aaduan-b] Profile: music preferences data structure
- [ ] [aaduan-b] Friend relationship data model
- [ ] [aaduan-b] Friend relationship management API (add / accept / decline)
- [ ] [jischoi] First-run account creation (email/password or social)
- [ ] [jischoi] Social account (Google or Facebook) linking after registration
- [ ] [jischoi] Mail validation on email/password signup
- [ ] [jischoi] Forgot password / reset flow
- [ ] [jischoi] Profile public info API (GET / PATCH)
- [ ] [jischoi] Profile friends-only info API (GET / PATCH)
- [ ] [jischoi] Profile private info API (GET / PATCH)
- [ ] [jischoi] Profile music preferences API (GET / PATCH)
- [ ] [aseisenb] Login / registration screen
- [ ] [aseisenb] Profile screen (read view UI + edit UI)
- [ ] [aseisenb] Friend management screen
- [ ] [aaduan-b] Tests: data models, friend relationship API
- [ ] [jischoi] Tests: authentication, profile CRUD API
- [ ] [aseisenb] Tests: UI components

### V.2.1 — Music Track Vote

- [ ] [aaduan-b] Users can suggest tracks for the current event playlist
- [ ] [aaduan-b] Users can vote on tracks; higher votes move tracks up the queue
- [ ] [aaduan-b] Visibility: Public by default; all users can find and vote
- [ ] [aaduan-b] Visibility: Private mode; only invited users can find and vote
- [ ] [aaduan-b] License Tier 1: everyone can vote (default)
- [ ] [aaduan-b] License Tier 2: invited users only can vote
- [ ] [aaduan-b] License Tier 3: users at a specific location within a specific time window can vote
- [ ] [aaduan-b] Concurrency handled correctly (simultaneous votes on same or different tracks)
- [ ] [aaduan-b] Tests: track suggestion, voting logic, license tiers, concurrency, visibility

### V.2.3 — Music Playlist Editor

- [ ] [jischoi] Real-time collaborative playlist editing (add / remove / reorder tracks)
- [ ] [jischoi] Visibility: Public by default; every user has access
- [ ] [jischoi] Visibility: Private mode; only invited users have access
- [ ] [jischoi] License Tier 1: everyone can edit (default)
- [ ] [jischoi] License Tier 2: invited users only can edit
- [ ] [jischoi] Concurrency handled correctly (simultaneous moves on same or different tracks)
- [ ] [jischoi] Tests: real-time editing, license tiers, concurrency, visibility

### V.3 — Server

> Principle: All service data must be stored server-side. The backend is the source of truth.
> Stack: Python + FastAPI + uv / PostgreSQL / WebSocket
> Local dev: Docker + uv / Deployment: Railway

- [ ] [aaduan-b] Technology choice justified and documented in `README`

### V.4 — API

> Principle: REST architecture and JSON exchange format are adopted. FastAPI provides Swagger UI at `/docs` automatically.

- [ ] [jischoi] API documentation auto-generated via FastAPI Swagger (`/docs`); verify all methods, inputs, and outputs are covered

### V.5 — Mobile Application

> Principle: The app acts only as a remote control; all logic lives in the backend.
> Stack: React Native (Expo) + React Native Web / Zustand / axios / expo-auth-session

- [ ] [aseisenb] Target platform chosen and documented in `README` (iOS + Android via Expo, with justification)
- [ ] [aseisenb] Backend address configurable within the app (for testing)
- [ ] [aseisenb] Social login (Google or Facebook) implemented via expo-auth-session
- [ ] [aseisenb] Tests: UI components, navigation, API integration

### V.6 — Security

- [ ] [jischoi] Authenticated users can access only their own data
- [ ] [jischoi] Brute-force protection implemented (rate limiting or equivalent)
- [ ] [jischoi] Additional threats identified and mitigations documented in `README`
- [ ] [jischoi] Every mobile app action generates a backend log (platform, device, app version)
- [ ] [Shared] All credentials and API keys stored in .env, excluded from git
- [ ] [jischoi] Tests: data isolation, rate limiting, logging

### V.7 — Ramp-up

- [ ] [-] Load test conducted (AB / Gatling / JMeter or equivalent)
- [ ] [-] Maximum simultaneous user capacity measured and documented in `README`
- [ ] [-] Server specifications documented in `README` (CPU, RAM, cloud or on-premise)

### V.8 — Agility / Quality / CI

- [ ] [jischoi] CI pipeline configured on GitLab (auto-run tests on MR to main/ `.gitlab-ci.yml`)
    - [ ] [jischoi] Protected main branch: requires 1 approval from a non-author before merge
- [ ] [All] Unit tests written and maintained per feature by each owner
- [ ] [Shared] Dependencies auto-installable from a fresh clone (Makefile or equivalent)
- [ ] [Shared] No third-party libraries committed directly to the repo

### VI.1 — Bonus: Multi-platform Support

- [ ] [aseisenb] Web responsive client covering all service screens

### VI.2 — Bonus: IoT / IBeacon

- [ ] [-] IBeacon proximity detection for nearby public events
- [ ] [-] Event-to-beacon linking and proximity notification API
- [ ] [-] Tests: beacon detection, event linking

### VI.3 — Bonus: Free vs. Paid Subscription

- [ ] [Shared] Free and Paid plan definitions established and documented in README
  - Track Vote
    - Free:
      - vote participation (public event)
      - participation in invited private event
    - Paid:
      - event creation
      - private event creation + invitation
      - License Tier 2 configuration
      - License Tier 3 configuration
  - Playlist Editor
    - Free:
      - personal playlist creation
      - participation in invited playlist editing
    - Paid:
      - view other users' playlists
      - edit permission sharing (invitation)
      - License Tier 2 configuration
- [ ] [Shared] Plan switching (upgrade / downgrade) implemented
- [ ] [aaduan-b] Track Vote paid-only features gated behind subscription check
- [ ] [aaduan-b] Tests: Track Vote plan gating
- [ ] [jischoi] Playlist Editor paid-only features gated behind subscription check
- [ ] [jischoi] Tests: Playlist Editor plan gating

### VI.4 — Bonus: Offline Mode

- [ ] [aseisenb] Key data cached locally for offline use
- [ ] [aseisenb] Offline UI / feature degradation handled
- [ ] [aaduan-b] Sync on reconnection with conflict resolution strategy
- [ ] [Shared] Stale cache invalidation on reconnect
- [ ] [aaduan-b] Tests: sync, conflict resolution
- [ ] [aseisenb] Tests: offline UI, local caching

---
# Task Table
## V.1 — User / Data Model

| # | Task | Details | Owner | Type |
|---|------|---------|-------|------|
| 1 | User model definition | Auth fields + profile fields; agreed upon jointly before all other work starts | Shared | Mandatory |
| 2 | Profile data structure design | Public / friends-only / private field separation | aaduan-b | Mandatory |
| 3 | Music preferences data model | Genres, artists, and related preference schema | aaduan-b | Mandatory |
| 4 | Friend relationship data model | Minimum implementation to determine friend status; required for friends-only profile visibility | aaduan-b | Mandatory |
| 5 | Friend relationship management API | Add / accept / decline, friend list retrieval; must be completed before jischoi can implement friends-only access control | aaduan-b | Mandatory |

---

## V.1 — User / Authentication & API

| # | Task | Details | Owner | Type |
|---|------|---------|-------|------|
| 6 | Email + password registration | Password hashing, input validation | jischoi | Mandatory |
| 7 | Email verification flow | Send verification email on signup, token validation endpoint | jischoi | Mandatory |
| 8 | Password reset flow | Send reset link by email, token expiry handling | jischoi | Mandatory |
| 9 | Google OAuth login / registration | Mobile app + backend token verification | jischoi | Mandatory |
| 10 | Facebook OAuth login / registration | Mobile app + backend token verification | jischoi | Mandatory |
| 11 | Link social account to existing account | Allow adding Google / Facebook to an already registered account | jischoi | Mandatory |
| 12 | Profile CRUD API (public / friends-only / private) | Enforce visibility rules per field group; depends on friend relationship API | jischoi | Mandatory |
| 13 | Music preferences CRUD API | Read / write music preference data | jischoi | Mandatory |

---

## V.1 — User / Frontend

> Use mock data until backend APIs are ready.

| # | Task | Details | Owner | Type |
|---|------|---------|-------|------|
| 14 | Login / registration screen | UI for email+password, Google, and Facebook options | aseisenb | Mandatory |
| 15 | Profile screen (read view UI + edit UI) | Separate tabs for public / friends-only / private / music preferences | aseisenb | Mandatory |
| 16 | Friend management screen | Send / accept / decline friend requests | aseisenb | Mandatory |

---

## V.2.1 — Music Track Vote

| # | Task | Details | Owner | Type |
|---|------|---------|-------|------|
| 17 | Event creation / update / deletion (Public / Private) | Default Public; manage invited user list when Private | aaduan-b | Mandatory |
| 18 | Event invitation management | Send invitations, accept flow, access control for Private events | aaduan-b | Mandatory |
| 19 | Track suggestion feature | Allow users to suggest tracks for the current playlist | aaduan-b | Mandatory |
| 20 | Track voting and real-time ranking | Auto-sort playlist by vote count, real-time update | aaduan-b | Mandatory |
| 21 | Concurrent vote conflict handling | Guarantee data consistency under simultaneous votes (transactions / locks) | aaduan-b | Mandatory |
| 22 | License Tier 1: default (everyone can vote) | No restrictions, default state | aaduan-b | Mandatory |
| 23 | License Tier 2: invited users only can vote | Vote permission based on invitation list | aaduan-b | Mandatory |
| 24 | License Tier 3: location + time-window voting | Validate GPS coordinates range and time window simultaneously | aaduan-b | Mandatory |
| 25 | Public event search / listing | API to browse Public events | aaduan-b | Mandatory |

---

## V.2.3 — Music Playlist Editor

| # | Task | Details | Owner | Type |
|---|------|---------|-------|------|
| 26 | Playlist creation / update / deletion (Public / Private) | Default Public; only invited users can access when Private | jischoi | Mandatory |
| 27 | Playlist invitation management | Send invitations and enforce access control | jischoi | Mandatory |
| 28 | Real-time track add / remove / reorder | Real-time sync via WebSocket or equivalent | jischoi | Mandatory |
| 29 | Concurrent edit conflict handling | Resolve conflicts when multiple users move the same or different tracks simultaneously | jischoi | Mandatory |
| 30 | License Tier 1: default (everyone can edit) | No restrictions, all users with access can edit | jischoi | Mandatory |
| 31 | License Tier 2: invited users only can edit | Invited users edit; others are read-only | jischoi | Mandatory |
| 32 | Public playlist browse / listing | API to list Public playlists | jischoi | Mandatory |

---

## V.3 / V.4 — Server & API

| # | Task | Details | Owner | Type |
|---|------|---------|-------|------|
| 33 | Backend setup | Python + FastAPI + uv; initialize with `uv init`, configure `pyproject.toml` | Shared | Mandatory |
| 34 | Docker local dev environment | `Dockerfile` + `docker-compose.yml` for local development; PostgreSQL container included | Shared | Mandatory |
| 35 | Database setup | PostgreSQL via Docker locally / Railway PostgreSQL plugin for deployment; configure SQLAlchemy 2.0 async connection | aaduan-b | Mandatory |
| 36 | Technology choice documented in README | FastAPI, PostgreSQL, WebSocket, Docker, Railway — justify each choice | aaduan-b | Mandatory |
| 37 | Swagger auto-documentation verification | Confirm all endpoints appear correctly in FastAPI `/docs` | jischoi | Mandatory |
| 38 | Makefile setup | `make dev` (Docker), `make run` (local), `make test` (pytest), `make deploy` (Railway) | Shared | Mandatory |
| 39 | JWT auth middleware | Apply to all protected endpoints via FastAPI dependency injection | jischoi | Mandatory |

---

## V.5 — Mobile Application (Frontend)

| # | Task | Details | Owner | Type |
|---|------|---------|-------|------|
| 40 | Mobile app setup (React Native + Expo) | Initialize Expo project; configure React Native Web for responsive support | aseisenb | Mandatory |
| 41 | Backend address configuration screen | Allow changing API base URL from within the app for testing | aseisenb | Mandatory |
| 42 | Login / registration screens | UI for email+password, Google, and Facebook options | aseisenb | Mandatory |
| 43 | Profile screen (read view UI + edit UI) | Separate tabs for public / friends-only / private / music preferences | aseisenb | Mandatory |
| 44 | Music Track Vote full UI | Event list, creation, voting, real-time ranking screens | aseisenb | Mandatory |
| 45 | Music Playlist Editor full UI | Playlist list, creation, real-time collaborative editing screens | aseisenb | Mandatory |
| 46 | Friend management UI | Minimum UI to add friends and determine friend status; required for friends-only profile visibility | aseisenb | Mandatory |
| 47 | Event invitation UI (Track Vote) | Invite users to private events, manage invite list | aseisenb | Mandatory |
| 48 | Playlist invitation UI (Playlist Editor) | Invite users to private playlists, manage invite list | aseisenb | Mandatory |


---

## V.6 — Security

| # | Task | Details | Owner | Type |
|---|------|---------|-------|------|
| 49 | Rate limiting (brute-force protection) | Limit attempt count on sensitive endpoints such as login | jischoi | Mandatory |
| 50 | User data isolation verification | Include tests confirming no cross-user data access | jischoi | Mandatory |
| 51 | Security threat documentation | Describe threats (session hijacking, MITM, etc.) and applicable mitigations | jischoi | Mandatory |
| 52 | Backend action logging | Record Platform, Device, and App Version for every mobile action | jischoi | Mandatory |
| 53 | .env file management and .gitignore setup | All API keys and credentials must be in environment variables, excluded from git | Shared | Mandatory |

---

## V.7 — Ramp-up

| # | Task | Details | Owner | Type |
|---|------|---------|-------|------|
| 54 | Load test conducted | Use AB / Gatling / JMeter or equivalent | - | Mandatory |
| 55 | Maximum simultaneous user capacity measured and documented | Document results and justification in README | - | Mandatory |
| 56 | Server specifications documented | Railway plan specs (RAM, vCPU, region); documented in README | - | Mandatory |

---

## V.8 — Agility / Quality / CI

| # | Task | Details | Owner | Type |
|---|------|---------|-------|------|
| 57 | GitLab CI pipeline setup | Auto-run all tests on every push and MR (`uv run pytest`) | jischoi | Mandatory |
| 58 | Protected main branch configuration | Require 1 approval from non-author before merge to main | jischoi | Mandatory |
| 59 | Per-feature test maintenance | Each owner writes and updates tests on every feature change | All | Mandatory |
| 60 | Makefile setup | `make dev` / `make run` / `make test` / `make deploy` all defined | Shared | Mandatory |
| 61 | No third-party libraries committed to repo | Use uv / npm only | Shared | Mandatory |

---

## VI.1 — Multi-platform Support

| # | Task | Details | Owner | Type |
|---|------|---------|-------|------|
| 62 | Web responsive client implementation | All service screens adapted for mobile and desktop via React Native Web | aseisenb | Bonus |

---

## VI.2 — IoT / IBeacon

| # | Task | Details | Owner | Type |
|---|------|---------|-------|------|
| 63 | IBeacon detection mechanism | Auto-detect nearby public events and receive event info | - | Bonus |
| 64 | IBeacon event registration and management API | Link beacon info to events, trigger proximity notifications | - | Bonus |
| 65 | IBeacon nearby event UI | Display incoming event info when beacon is detected | - | Bonus |
| 66 | Tests: beacon detection, event linking | — | - | Bonus |

---

## VI.3 — Free vs. Paid Subscription

| # | Task | Details | Owner | Type |
|---|------|---------|-------|------|
| 67 | Subscription plan definition (Free / Paid) | Define and document feature restrictions per plan in README | Shared | Bonus |
| 68 | Plan switching implementation | Upgrade / downgrade flow | Shared | Bonus |
| 69 | Track Vote paid-only feature access control | Gate Track Vote paid features behind subscription check | aaduan-b | Bonus |
| 70 | Playlist Editor paid-only feature access control | Gate Playlist Editor paid features behind subscription check | jischoi | Bonus |
| 71 | Subscription plan UI | Plan comparison screen, upgrade / downgrade flow UI | aseisenb | Bonus |
| 72 | Paywall / locked feature UI | Display upgrade prompt when accessing paid-only features | aseisenb | Bonus |

---

## VI.4 — Offline Mode

| # | Task | Details | Owner | Type |
|---|------|---------|-------|------|
| 73 | Offline local data caching | Store key data (playlists, etc.) locally on device | aseisenb | Bonus |
| 74 | Offline UI / feature branching | Detect network state and provide degraded UX accordingly | aseisenb | Bonus |
| 75 | Sync on reconnection | Include conflict resolution strategy (last-write-wins, etc.) | aaduan-b | Bonus |
| 76 | Stale cache invalidation | Refresh local data against server state on reconnect | Shared | Bonus |
| 77 | Tests: sync, conflict resolution | — | aaduan-b | Bonus |
| 78 | Tests: offline UI, local caching | — | aseisenb | Bonus |
