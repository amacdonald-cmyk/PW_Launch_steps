# SSO Rollout — Order of Operations

> Migration from Omeda PW auth to TNTN token-based SSO across all platforms.
> Phases 1–3 are sequential prerequisites. Phase 4 launches both streams in parallel.

---

```
┌─────────────────────────────────────────────────────────────────┐
│  PHASE 1 — User migration                                       │
│  Migrate all Omeda PW auth users to TNTN token.                 │
│  Passwords are preserved. No user action required.              │
│                          [backend only]                         │
└───────────────────────────────┬─────────────────────────────────┘
                                │
                                ▼
┌─────────────────────────────────────────────────────────────────┐
│  PHASE 2 — Form updates                                         │
│  All active login and registration forms updated to issue and   │
│  validate TNTN tokens. PW token checks removed.                 │
│                          [before launch]                        │
└───────────────────────────────┬─────────────────────────────────┘
                                │
                                ▼
┌─────────────────────────────────────────────────────────────────┐
│  PHASE 3 — User communication                                   │
│  Users with both a PW token and TNTN token receive an email     │
│  heads-up: they may need to reset their password if they        │
│  experience any login issues after the transition.              │
└───────────────────────┬─────────────────────────────────────────┘
                        │
           ┌────────────┴────────────┐
           │     Phase 4 unlocks     │
           │          both           │
           ▼                         ▼
┌──────────────────────┐   ┌─────────────────────────┐
│  PHASE 4a            │   │  PHASE 4b               │
│  New order forms     │   │  Archive website        │
│                      │   │                         │
│  Launch new forms    │   │  New site launches      │
│  built natively on   │   │  with TNTN auth         │
│  TNTN auth.          │   │  integrated.            │
└──────────────────────┘   └─────────────────────────┘
```

---

## Phase details

### Phase 1 — User migration
- Batch migrate all Omeda customer records that have a PW auth token to also carry a TNTN token
- Passwords are **not** changed or invalidated
- This is a backend-only operation with no user-facing impact
- **Dependency:** Must complete before Phase 2 begins

### Phase 2 — Form updates
- All active order forms, login forms, and registration flows updated to:
  - Issue a TNTN token on successful auth (not a PW token)
  - Validate against TNTN token (not PW token)
- Edge case handling required for users with only a PW token (graceful fallback → password reset prompt)
- **Dependency:** Phase 1 complete

### Phase 3 — User communication
- Identify users who hold **both** a PW token and a TNTN token (transition-state users)
- Send proactive email: "You may need to reset your password if you experience trouble logging in"
- Email sets expectations before new forms go live — protects purchase experience
- **Dependency:** Phase 2 complete

### Phase 4 — Launch (parallel streams)

#### 4a — New order forms
- New forms built TNTN-native from scratch (not retrofitted)
- Handles three user states cleanly:
  - Existing user with TNTN token → straight through, no friction
  - Net-new user → account creation + TNTN token issued as part of order flow
  - Edge case PW-only user → graceful fallback with password reset prompt

#### 4b — Archive website
- New Archive site launches with TNTN auth integrated end-to-end
- Wristband access tiers apply: access level determined by subscription status at this property
- No Omeda auth dependency at the front-end layer

---

## Token state reference

| User state | PW token | TNTN token | Behavior after launch |
|---|---|---|---|
| Migrated existing user | ✅ | ✅ | Receives Phase 3 email. Logs in via TNTN. |
| New user (post-launch) | ❌ | ✅ | TNTN issued at registration. No PW token created. |
| Edge case legacy user | ✅ | ❌ | Fallback prompt to reset password. |
| Fully migrated user | ❌ | ✅ | Standard TNTN login. No action needed. |

---

*Part of the SSO platform rollout. See also: [SSO architecture overview](./sso-architecture.md)*
