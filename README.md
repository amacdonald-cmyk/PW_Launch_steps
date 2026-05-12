# PW Launch Steps

Internal documentation for the rollout of our SSO (Single Sign-On) platform — migrating authentication across all partner properties from Omeda PW token auth to a TNTN token-based system built on a Cloudflare Worker API.

---

## What this repo is

This repo tracks the planning, architecture, and order of operations for the SSO platform launch. It is intended for engineering, product, and stakeholder reference.

> **Status:** Pre-launch — Phases 1–3 in progress

---

## Quick links

| Document | Description |
|---|---|
| [SSO Rollout — Order of Operations](./sso-rollout-phases.md) | Phase-by-phase launch plan with dependencies and open questions |
| [SSO Architecture Overview](./sso-architecture.md) | How the platform works — components, auth flow, and access tiers |

---

## The short version

We are replacing Omeda-based authentication with an in-house SSO layer. Users log in once and receive a signed TNTN token (wristband) that is trusted across all partner properties. Access level at each property is determined by the user's subscription status there — not by the token alone.

Omeda remains the system of record for customer data, subscriptions, and transactions. The SSO layer sits in front of it, handling authentication independently so we are not coupled to Omeda's auth infrastructure.

---

## Why now

- Omeda auth ties our login experience to a vendor — any negotiation, migration, or platform addition puts user login at risk
- We need cross-platform SSO capability our partner sites can rely on
- Owning the auth layer means we can migrate the back-end (Omeda or otherwise) without users ever feeling it

---

## Repo structure

```
PW_Launch_steps/
├── README.md                  ← You are here
├── sso-rollout-phases.md      ← Launch order of operations
└── sso-architecture.md        ← Platform architecture and auth flow
```

---

## Rollout phases at a glance

| Phase | What happens | Dependency |
|---|---|---|
| 1 — User migration | Migrate all Omeda PW auth users to TNTN tokens | — |
| 2 — Form updates | Active forms updated to issue and validate TNTN tokens | Phase 1 |
| 3 — User comms | Email users with dual tokens: heads-up on possible password reset | Phase 2 |
| 4a — New order forms | Launch new forms built natively on TNTN auth | Phase 3 |
| 4b — Archive website | Launch Archive site with TNTN auth integrated | Phase 3 |

See [sso-rollout-phases.md](./sso-rollout-phases.md) for full detail and open questions.

---

## Contact

Repo maintained by `amacdonald-cmyk`. Questions on the SSO platform architecture or launch timeline should be directed to the product team.
