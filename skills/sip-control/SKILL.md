---
name: sip-control
description: Control a Dan-in-CA/SIP sprinkler/irrigation controller over its HTTP API. Use when the user wants to check SIP status, turn stations on/off, run a program, run-once, set rain delay, pause, or otherwise drive their SIP Raspberry Pi irrigation controller programmatically.
---

# SIP Control (Dan-in-CA/SIP)

SIP (Sustainable Irrigation Platform, github.com/Dan-in-CA/SIP) exposes a
web.py HTTP API. ALL control endpoints are `GET` requests. This is verified
against `urls.py` and `webpages.py` in the SIP master branch.

## Setup
Set these environment variables before use:
  - `SIP_BASE_URL` — controller base URL, e.g. `http://<host>:8080`
  - `SIP_PASSWORD` — plaintext password; omitted entirely if auth is disabled on the device

## Authentication (verified from helpers.check_login)
- SIP has a "use password" flag `upas`. If `upas == 0` -> NO auth required.
- Otherwise pass the **plaintext** password as `?pw=PASSWORD` on any endpoint.
  The server computes `sha256(pw)` itself, so do NOT pre-hash it.
- Session-cookie login also works: `POST /login` with form field `password`.
  Endpoints that require login are subclasses of `ProtectedPage`.

## Endpoints (verified)
| Method | Path | Purpose | Key params |
|--------|------|---------|------------|
| GET | `/api/status` | JSON status of all stations + system | — |
| GET | `/api/log` | JSON run log | — |
| GET | `/api/plugins` | plugin data | — |
| GET | `/sn` | get/set a station | `sid` (1-based; 0=all), `set_to` (1/0), `set_time` (sec; 0=until off). **Requires manual mode (`mm=1`).** |
| GET | `/cr` | run-once program | `t=` JSON array of per-station seconds, 0-based, e.g. `t=[0,300,0]` |
| GET | `/rp` | run a saved program now | `pid` (0-based program index) |
| GET | `/cv` | change controller values | **`mm=1` → Manual mode, `mm=0` → Auto** (Auto calls clear_mm → stops manual stations); `en=0` (all off), `rd=<hours>` (rain delay), `rsn=1` (pause/rain stop) |
| GET | `/ep` | enable/disable program | `pid`, `enable` (1/0) |
| GET | `/dp` | delete program | `pid` (`-1` = all) |
| GET | `/cp` | add/modify program | `pid`, `v=` JSON program object |
| GET | `/restart` | restart SIP | — |

## Operational SOP (user conventions)
When the user gives verbal commands, switch the Manual/Auto mode FIRST, then act:
- **"oprește stațiile" (stop the stations)** → `GET /cv?mm=1` (Manual) first, then stop
  (e.g. `GET /cv?en=0` for all-off, or `GET /cv?rsn=1` to pause).
- **"pornesc" / "vreau să le pornesc" (start them)** → `GET /cv?mm=0` (Auto) first,
  then start (e.g. `GET /rp?pid=N`, or let the schedule run).
Do NOT skip the mode switch — Manual is expected for stopping, Auto for starting.
```

## Pitfalls
- `/sn` `set_to` returns "Manual mode not active." unless manual mode is on
  (`/cv?mm=1`). Use `/cr` (run-once) to avoid that requirement.
- `/cr?t=` is a 0-based JSON array indexed by station. Pad to the number of
  stations; wrong index waters the wrong zone.
- Active relay polarity is set by the `relay_board` plugin (`active: low/high`),
  independent of these endpoints — wiring-side, not API-side.
- Default HTTP port is **8080** (option `htp`); adjust if behind a proxy.
- Always `GET /api/status` first to confirm state before/after a write op.
