# SSO Architecture Overview

How the TNTN token-based Single Sign-On platform works — components, auth flow, access tiers, and how it relates to Omeda.

---

## The building analogy

Think of it as a key card system across a campus of buildings.

- **Building entrances** = partner websites (e.g. popularwoodworking.com)
- **Central security desk** = SSO server (Cloudflare Worker API)
- **Records room** = Omeda (identity and subscription data)
- **Wristband** = TNTN token (digitally signed, trusted by all entrances)
- **Wristband color** = access tier (determined per property by subscription status)

---

## Components

| Component | What it is | Role |
|---|---|---|
| Partner sites | e.g. popularwoodworking.com | Building entrances — redirect unauthenticated users to SSO |
| React SPA | Login and account pages | Sign-in booth — collects credentials |
| Cloudflare Worker API | Edge API layer | Desk clerk — orchestrates auth, issues tokens |
| Cloudflare KV | Key-value session store | Wristband locker — holds active sessions, tokens, auth codes |
| D1 Database | SQLite activity log | Entry log — records all auth events |
| Omeda Authentication API | Credential validation | Confirms identity against master records |
| Omeda Customer API | Subscription and entitlement data | Determines what the user is entitled to access |
| JWKS endpoint | RS256 public key | Allows partner sites to verify token signatures without calling Omeda |

---

## Auth flow

```
Partner site
    │
    │  1. User arrives — redirected to login with return_url + state
    ▼
React SPA (login page)
    │
    │  2. POST /api/auth — email + password submitted
    ▼
Cloudflare Worker API
    │
    ├──► 3. Omeda Authentication API — validate credentials
    │         │
    │         ▼
    │    4. OmedaCustomerId returned
    │
    ├──► 5. Omeda Customer API — fetch entitlements
    │         │
    │         ▼
    │    Fresh entitlements returned
    │
    │  5. Session + auth code created → stored in Cloudflare KV
    │  6. Activity logged → D1 Database
    │
    │  7. Redirect back to partner site with code + state
    ▼
Partner site
    │
    │  8. POST /api/token — exchange code for JWT + refresh token
    ▼
Cloudflare Worker API
    │
    │  9. Code consumed, refresh token rotated in KV
    │
    │  11. Return JWT + refresh token to partner site
    ▼
Partner site
    │
    │  Verify JWT signature via JWKS endpoint (RS256 public key)
    │  No further Omeda calls needed for session duration
    ▼
User is in — access level applied based on entitlements
```

### Token refresh

When a JWT expires, the partner site calls `POST /api/refresh`. The Worker rotates the refresh token in KV and returns a new JWT. This happens in the background — the user is never logged out mid-session.

---

## Access tiers

The same TNTN token works at every partner property. Each property independently determines the user's access level based on their subscription status at that site.

| Tier | Who | Access |
|---|---|---|
| 🥇 Gold | Full subscriber at this property | All areas — full content access |
| 🥈 Silver | Registered user, no active subscription | Lobby access — free and preview content only |
| 🚪 No pass | Unknown visitor | Front door only — must register or subscribe |

A gold subscriber at Property A arriving at Property B gets silver access at B unless they hold a subscription there too. The token proves identity — the property decides access.

---

## How this differs from Omeda-based auth

| | Omeda-based auth | TNTN SSO platform |
|---|---|---|
| Login experience | Separate per site | One login across all properties |
| Vendor dependency | Fully coupled to Omeda | Omeda stays in back-end only |
| Ability to migrate | Disrupts user logins | Invisible to users |
| Cross-platform access | Limited by Omeda's capabilities | Built to our spec |
| Activity visibility | Omeda reporting only | Full log we own (D1) |
| Access tiers | Binary | Tiered per property per subscriber |
| Token refresh | Session managed by Omeda | Silent background refresh via KV |

---

## Why Omeda is not being replaced

Omeda remains the system of record for:
- Customer identity and profile data
- Subscription transactions and billing
- Entitlement records

The SSO layer calls Omeda at login to verify identity and fetch entitlements. After that, the JWT carries what the system needs for the session. Omeda is never called again until the next full login or token refresh requiring re-validation.

This means: if we ever need to migrate off Omeda, swap in a different entitlement system, or renegotiate the contract — users never see it. The wristband system stays identical. Only what's behind it changes.

---

## Security notes

- JWTs are signed with RS256 — partner sites verify signatures using the public JWKS endpoint without needing access to the private key
- Auth codes are single-use and consumed on exchange
- Refresh tokens are rotated on every use — a replayed token is invalid
- All auth activity is logged to D1 for audit purposes
- Sessions and tokens are stored in Cloudflare KV with TTLs

---

*See also: [SSO Rollout — Order of Operations](./sso-rollout-phases.md)*
