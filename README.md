# BLEEDTHROUGH
### Agent Failure Series #20

> **What happens when your AI support agent serves two tenants at once — and session isolation fails?**

One user's name, email, and credit card number bleeds into another user's conversation. The agent doesn't notice. It doubles down.

---

## What It Shows

A live, interactive simulator of **multi-tenant context isolation failure** in a shared AI support agent.

Under load, cache key collisions cause User B's session to read User A's data:
- `alice@acme.com` → appears in Bob's chat
- Credit card last4: `4242` → shown to the wrong customer
- The agent confidently **doubles down** when Bob reports the discrepancy

---

## Try It

🔴 **[Live Demo → rlasaf12.github.io/bleedthrough](https://rlasaf12.github.io/bleedthrough/)**

1. Click **Start Sessions** — Alice, Bob, Carol open concurrent support chats
2. Click **Apply Load → Trigger Bleed** — watch cache collision happen in real time
3. Toggle **Protected Mode** — see how tenant namespace isolation blocks the bleed
4. Watch the **Event Log** and **Stats Panel** for incident timeline

---

## The Failure Mode

```
Load spike: 900 req/min
↓
Cache key: session_{user_id}  ← no tenant namespace
↓
Collision: Bob's lookup hits Alice's cache slot
↓
Agent reads: alice@acme.com | Acme Corp | Enterprise | card:4242
↓
Agent responds to Bob with Alice's data
↓
Bob: "That's not my info"
↓
Agent: *doubles down* (still reading wrong cache)
```

### Root Cause

Shared Redis cache with no per-tenant namespace. Under load, key collisions are probabilistic but real. Without namespace enforcement, there's no runtime signal — the agent reads stale data confidently.

---

## The Fix (5 Controls)

| Defense | What It Does |
|---------|-------------|
| **Tenant namespace** | `tenant_{tid}:session_{uid}` — eliminates collisions |
| **TTL enforcement** | Short expiry limits blast radius |
| **Strict deserialization** | Type-check cache reads before use |
| **Tenant context assertion** | Runtime guard: verify tenant before every response |
| **Blind spot audit** | Scan all cache reads for namespace gaps |

---

## Series Context

| # | Name | Failure Mode |
|---|------|-------------|
| 18 | GRIDLOCK | Deadlock in multi-agent coordination |
| 19 | ORPHANCALL | Async call fired and forgotten |
| **20** | **BLEEDTHROUGH** | **Multi-tenant context isolation failure** |

**Full series:** [github.com/RLASAF12](https://github.com/RLASAF12?tab=repositories)

---

## Stack

Single-file HTML · No backend · No API calls · No credentials · Fully self-contained

Built by [Ben](https://github.com/RLASAF12) — the nightly prototype builder in Harel's AI agent team.
