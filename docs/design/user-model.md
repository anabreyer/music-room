# User Model — Design Spec (Task #1)

> **Status:** Draft for agreement
> **Owners:** aaduan-b (data model) + jischoi (auth) — *must be jointly agreed before any other V.1 work starts*
> **Subject refs:** V.1 (User), V.5 (social login), V.6 (security)
> **Stack:** PostgreSQL · SQLAlchemy 2.0 (async) · FastAPI · JWT · Expo + expo-auth-session

This document is the contract between the **data model** (aaduan-b) and the **authentication / API** (jischoi) work. It defines: what we collect, how users log in, how we authenticate them, how profile visibility works, and the exact table schema. Everything downstream (profile APIs, friend relationships, services) builds on the tables defined here.

---

## 1. Locked decisions

| Decision | Choice | Rationale |
|---|---|---|
| Identity | Unique immutable **`@handle`** + free-text **`display_name`** | Friend search by handle, no email leakage |
| Email verification | **Block all access until verified** (email/password accounts) | Simplest + most secure; no per-endpoint verified checks |
| 2FA | **Schema fields reserved now, flows built later** | Beyond mandatory scope; don't block the critical path |
| Password hashing | **Argon2id** | Modern, memory-hard, OWASP-recommended |
| Auth tokens | **JWT** (short-lived access + rotating refresh) | Stateless, mobile-friendly (per Contributing) |
| Social providers | **Google + Facebook** via OAuth | Subject V.1 / V.5 |

---

## 2. Registration — what we ask for

A user registers **either** by email/password **or** by a social account (Google/Facebook). We collect the **minimum** at registration and let users complete their profile afterward.

### 2.1 Email / password registration
Collected at signup:

| Field | Required | Notes |
|---|---|---|
| `email` | ✅ | Unique, case-insensitive, validated format |
| `password` | ✅ | Plain text only in transit (HTTPS); stored as Argon2id hash. Min 8 chars, strength rules enforced by jischoi |
| `handle` | ✅ | Unique `@handle`, 3–30 chars, `[a-z0-9_]`, case-insensitive unique |
| `display_name` | ✅ | Free text, 1–50 chars |

→ Account created with `email_verified = false`. A verification email is sent. **The user cannot log in / call the API until they verify** (decision §1).

### 2.2 Social registration (Google / Facebook)
The provider returns a verified email + a stable provider user id. We:
1. Create a `users` row (`password_hash = NULL`, `email_verified = true` — the provider already verified it).
2. Create an `auth_identities` row linking the provider.
3. Ask the user to pick a `handle` + `display_name` on first run (the only fields the provider can't give us reliably).

→ A social-only account has **no password**. They can later "add a password" to enable email/password login too.

### 2.3 What we deliberately do *not* ask at registration
Real name, birthday, location, avatar, music preferences — all optional and edited later from the profile (keeps signup friction low; these live in the profile field groups in §6).

---

## 3. Login methods

| Method | Flow |
|---|---|
| Email + password | `POST /auth/login` → verify Argon2id hash → issue JWT pair. Rejected if `email_verified = false`. |
| Google | Client gets Google token via `expo-auth-session` → backend verifies token → matches `auth_identities` → issues JWT pair. |
| Facebook | Same as Google. |

All three converge on the same output: an **access token + refresh token** (§4). The mobile app stores them and is just a remote control (subject V.5).

---

## 4. Authentication mechanism (JWT)

> Implementation owned by **jischoi** (task #39 JWT middleware). Defined here so the model supports it.

- **Access token** — short-lived (~15 min), stateless, carries `sub = user.id`. Sent as `Authorization: Bearer`. Not stored server-side.
- **Refresh token** — long-lived (~30 days), **stored hashed** in a `sessions` table so it can be revoked (logout, theft). Rotated on each use.
- The `sessions` table also captures the **device context** the subject (V.6) requires logging for every action: platform, device, app version.

```
access  = JWT{ sub, exp, iat, type:"access" }      # not persisted
refresh = opaque random → SHA-256 stored in sessions  # revocable
```

This gives us logout, "log out all devices", session-theft revocation, and the V.6 action log in one place.

---

## 5. Account & auth identity model

We separate **who the user is** (`users`) from **how they can authenticate** (`auth_identities`). This cleanly supports: register-with-social, link-social-to-existing, and multiple providers per account.

```
users ─┬─< auth_identities      (0..N linked Google/Facebook providers)
       ├─< sessions             (active refresh tokens / devices)
       └──  password_hash       (nullable — inline on users)
```

A user can authenticate via **any** of: their password (if set) + each linked social identity. Linking (task #11) = inserting an `auth_identities` row on the already-authenticated user. Unlinking is allowed only if the user still has at least one usable login method (password or another identity) — never lock yourself out.

---

## 6. Profile fields & visibility

The subject requires four field groups: **public**, **friends-only**, **private**, and **music preferences**. We model visibility with a single convention so the profile API (jischoi task #12) can enforce it uniformly.

> Detailed profile *structure* is aaduan-b task #2; music preferences schema is task #3. This section fixes the **groups, the default home of each core field, and the enforcement contract** so those tasks have a stable base.

### 6.1 Visibility levels
```python
class Visibility(enum.Enum):
    PUBLIC  = "public"   # anyone authenticated
    FRIENDS = "friends"  # accepted friends only (friend model = aaduan-b task #4)
    PRIVATE = "private"  # the user themselves only
```

### 6.2 Core field placement

| Field | Group | Notes |
|---|---|---|
| `handle` | public | identity, immutable |
| `display_name` | public | editable |
| `avatar_url` | public | editable |
| `bio` | public | editable, short text |
| `real_name` | friends-only* | optional |
| `city` / coarse location | friends-only* | optional; **not** precise GPS |
| `music_preferences` | configurable* | genres/artists — task #3; default friends-only |
| `email` | private | never exposed to other users |
| `date_of_birth` | private | optional |
| `linked_providers` | private | which socials are linked |

\* aaduan-b task #2 may make per-field visibility user-configurable (`PUBLIC/FRIENDS/PRIVATE` per field). For the model, store a **visibility setting per overridable field** (e.g. a `profile_field_visibility` JSON/table) so users can "state and update" visibility as the subject demands. Fixed-group fields (handle public, email private) are not overridable.

### 6.3 Enforcement contract (for jischoi's profile API)
When user **A** requests user **B**'s profile, the API returns a field iff:
- field group is `public`, **or**
- group is `friends` **and** A and B are accepted friends, **or**
- group is `private` **and** A == B.

`GET /users/{handle}` returns only the visible subset for the requester. `PATCH /users/me` lets a user update their own fields + their per-field visibility.

---

## 7. Schema (SQLAlchemy 2.0 sketch)

> Reference shape for the agreement — final migrations live with aaduan-b (task #35). Uses `citext` for case-insensitive unique email/handle.

```python
class User(Base):
    __tablename__ = "users"

    id:            Mapped[uuid.UUID] = mapped_column(primary_key=True, default=uuid.uuid4)
    handle:        Mapped[str]  = mapped_column(CITEXT, unique=True, index=True)   # @handle
    display_name:  Mapped[str]  = mapped_column(String(50))
    email:         Mapped[str]  = mapped_column(CITEXT, unique=True, index=True)
    email_verified: Mapped[bool] = mapped_column(default=False)
    password_hash: Mapped[str | None] = mapped_column(String(255))                # NULL = social-only

    # profile
    avatar_url:    Mapped[str | None]
    bio:           Mapped[str | None] = mapped_column(String(300))
    real_name:     Mapped[str | None]
    city:          Mapped[str | None]
    date_of_birth: Mapped[date | None]
    # per-field visibility overrides for the configurable fields
    field_visibility: Mapped[dict] = mapped_column(JSONB, default=dict)

    # --- 2FA: reserved, flows built later (decision §1) ---
    totp_secret:   Mapped[str | None] = mapped_column(String(255))   # encrypted at rest when enabled
    totp_enabled:  Mapped[bool] = mapped_column(default=False)

    status:        Mapped[str]  = mapped_column(String(20), default="active")  # active|suspended|deleted
    created_at:    Mapped[datetime] = mapped_column(default=func.now())
    updated_at:    Mapped[datetime] = mapped_column(default=func.now(), onupdate=func.now())

    identities:    Mapped[list["AuthIdentity"]] = relationship(back_populates="user")
    sessions:      Mapped[list["Session"]]      = relationship(back_populates="user")


class AuthIdentity(Base):              # linked social logins (Google/Facebook)
    __tablename__ = "auth_identities"
    id:               Mapped[uuid.UUID] = mapped_column(primary_key=True, default=uuid.uuid4)
    user_id:          Mapped[uuid.UUID] = mapped_column(ForeignKey("users.id", ondelete="CASCADE"))
    provider:         Mapped[str]  = mapped_column(String(20))      # "google" | "facebook"
    provider_user_id: Mapped[str]  = mapped_column(String(255))     # stable subject from provider
    created_at:       Mapped[datetime] = mapped_column(default=func.now())
    user:             Mapped["User"] = relationship(back_populates="identities")
    __table_args__ = (UniqueConstraint("provider", "provider_user_id"),)


class Session(Base):                   # refresh tokens + device log (V.6)
    __tablename__ = "sessions"
    id:                 Mapped[uuid.UUID] = mapped_column(primary_key=True, default=uuid.uuid4)
    user_id:            Mapped[uuid.UUID] = mapped_column(ForeignKey("users.id", ondelete="CASCADE"))
    refresh_token_hash: Mapped[str] = mapped_column(String(255), index=True)
    platform:           Mapped[str | None]   # "ios" | "android" | "web"
    device:             Mapped[str | None]   # "iPhone 14", ...
    app_version:        Mapped[str | None]
    expires_at:         Mapped[datetime]
    revoked_at:         Mapped[datetime | None]
    created_at:         Mapped[datetime] = mapped_column(default=func.now())
    user:               Mapped["User"] = relationship(back_populates="sessions")
```

**Tables owned by jischoi** (referenced, not defined here): `email_verification_tokens`, `password_reset_tokens` — both single-use, time-limited, FK → `users.id`. They flip `users.email_verified` / update `users.password_hash` respectively.

---

## 8. Security notes (V.6)

- Passwords: **Argon2id** only; never logged, never returned.
- Email/handle uniqueness is case-insensitive (`citext`) to prevent duplicate-identity tricks.
- Refresh tokens stored **hashed**; rotation on use detects theft (reuse of a rotated token ⇒ revoke the whole chain).
- Login is rate-limited (jischoi task #49) — model supports it via `sessions`/audit.
- A user can only ever read/write **their own** `users` row except for the visibility-filtered public view (§6.3) — this is the V.6 data-isolation guarantee at the model level.
- `password_hash = NULL` is a valid state (social-only) — auth code must branch on it, never assume a password exists.

---

## 9. Ownership split (so work can start in parallel)

| Area | Owner | Tasks |
|---|---|---|
| `users`, `auth_identities`, `sessions` migrations + profile/visibility structure | **aaduan-b** | #2, #3, #35 |
| Friend model (drives `FRIENDS` visibility) | **aaduan-b** | #4, #5 |
| Registration, email verify, password reset, OAuth, account linking, JWT, profile API | **jischoi** | #6–#13, #39 |
| 2FA flows | later | — |

---

## 10. Agreement checklist — sign off before V.1 work starts

- [ ] jischoi agrees on `users` / `auth_identities` / `sessions` shape
- [ ] jischoi agrees email-verified gate = hard block (no limited mode)
- [ ] jischoi agrees access/refresh token split + `sessions` as the revocation/device-log home
- [ ] aaduan-b owns `field_visibility` mechanism; jischoi's profile API consumes it (§6.3 contract)
- [ ] handle rules confirmed (`[a-z0-9_]`, 3–30, case-insensitive unique)
- [ ] 2FA fields reserved but no flow this milestone
