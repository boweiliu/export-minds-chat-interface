---
title: export minds chat interface
description: A REST API (plus a demo web UI) to list, message, and read a minds workspace's agents from outside.
thumbnail: inspiration-export-minds-chat-interface.svg
format: v1
---

# export minds chat interface

This file is the manifest for the **export minds chat interface** inspiration (slug:
`export-minds-chat-interface`). It is the one document a future agent reads to understand,
present, and adapt this inspiration. If you are an agent in a mind that was
created from this inspiration, this file is your script: read all of it, then
follow "How to adapt it" below.

## What it is

A REST API (plus a demo web UI) to list, message, and read a minds workspace's agents from outside.

A minds workspace's agents normally only take instructions through the built-in
chat UI, so anything outside the workspace -- your other agents, a script, your
phone -- has no clean way to reach them. This inspiration solves that by
"exporting" the chat interface as a small, token-authenticated **REST API**.
With one bearer token, an outside caller can **list** the workspace's agents,
**send a message into any agent's live session**, and **read any conversation
back** -- full history plus a live Server-Sent-Events stream. The headline
product is that agentic API (clean REST + SSE, an `llms.txt` guide you hand to
an agent, and a zero-dependency Python client). To make it tangible for people,
it also ships a **demo web console and landing page** that are just another
client of the *same* API: a left rail listing every agent with a live status
dot (idle / thinking / running a tool), the selected conversation in the main
pane, and a box to send a message. When it is running, the user sees that web
console as a workspace tab, and -- because a dedicated public tunnel exposes the
token-gated bridge -- a shareable public URL (with a QR code on the landing
page) they can open from a phone.

## How it works

The snapshot includes these paths (each is a repo-root-relative path copied
from the original mind onto a clean default-workspace-template base):

- `libs/chat_bridge`

`libs/chat_bridge` is the entire feature -- a single Python lib holding both
services, the web UI, the client, and docs:

- `src/chat_bridge/runner.py` -- the Flask entrypoint (`chat-bridge` console
  script) that assembles and serves the app.
- `src/chat_bridge/api.py` -- the REST + SSE routes: `GET /api/agents`,
  `POST /api/agents/<id-or-name>/message`, `GET /api/agents/<id-or-name>/events`
  (history), and `GET /api/agents/<id-or-name>/stream` (live SSE).
- `src/chat_bridge/upstream.py` -- the client for the workspace's own
  `system_interface` HTTP API on loopback (`http://127.0.0.1:8000`); it fetches
  the agent list and transcripts and derives each agent's activity status
  (idle / thinking / running a tool) from the tail of its transcript.
- `src/chat_bridge/auth.py` -- the bearer-token wall (constant-time comparison)
  for API callers plus the cookie-based sign-in the web UI uses; the token is
  auto-generated on first boot into `runtime/secrets/chat_bridge.env`.
- `src/chat_bridge/assets/index.html` -- the demo web console + landing page,
  a pure client of the same REST API (it authenticates with a cookie, renders
  the agent rail, the conversation, the send box, `llms.txt`, and the public
  URL with a QR code).
- `src/chat_bridge/tunnel.py` -- the `chat-bridge-tunnel` console script, an
  account-less Cloudflare quick tunnel pointed only at port 8082; it writes the
  assigned public URL to `runtime/chat-bridge/public_url.txt` for the landing
  page.
- `src/chat_bridge/client.py` -- a zero-dependency Python client for the API.

At runtime, two supervisord programs (added to `supervisord.conf`) run the
feature. `chat-bridge` registers port **8082** with the workspace UI via
`scripts/forward_port.py --url http://localhost:8082 --name chat-bridge`, then
runs `uv run chat-bridge`; the Flask app listens on 8082 and proxies to the
workspace's `system_interface` API on loopback 8000, adding the token wall in
front. `chat-bridge-tunnel` runs `uv run chat-bridge-tunnel`, which starts the
Cloudflare quick tunnel aimed only at 8082 and publishes the resulting URL to
`runtime/chat-bridge/public_url.txt`, which the landing page reads. So only the
token-gated bridge (8082) is ever exposed publicly -- never the raw loopback
API on 8000.

## Prerequisites

Activation requirements: what the adopting agent must SET UP -- and must
INITIATE ITSELF during setup, before asking how to adapt -- for this
inspiration to run against the new user's own accounts/data. One line per
requirement, in this machine-readable form (greppable by `requires_`):

No prerequisites -- runs with no external permissions or secrets.

There is nothing for the adopting agent to set up in the `requires_permission:`
/ `requires_secret:` sense. The bridge needs no external-service credentials and
no latchkey permission grants: it generates its own bearer token on first boot
into `runtime/secrets/chat_bridge.env`, reads only the workspace's own loopback
API on `127.0.0.1:8000`, and the public URL comes from the bundled
account-less Cloudflare quick tunnel (`cloudflared`), which needs no Cloudflare
account or login. Just start the two services and it works against the
workspace it is running in.

## How to adapt it

Instructions for the NEXT agent -- the one adapting this inspiration into a
new mind. This is the `use-inspiration` skill's template path; in short:

1. Read this entire file first, especially "Prerequisites" and "Holes"
   below -- Prerequisites are your SETUP agenda, Holes are your ADAPTATION
   agenda.
2. Present the inspiration to the user in plain, non-technical language: what
   it is, what it does, and what it needs from them (name the Prerequisites).
3. Ask whether they want to use the same connectors (e.g. their own Slack).
   If YES: ACTIVATE FIRST -- initiate every `requires_permission` line NOW
   via a latchkey permission request (see the `latchkey` skill; the request
   opens the approval/login flow in the minds app), wire up any
   `requires_secret` values, start the services, and get the app showing
   THE USER'S OWN DATA. Done for a data-backed app means the user can open it
   and see their own data -- NOT that a service starts or an endpoint returns
   200. Then tell them it is live and to take a look.
4. Only AFTER that (or immediately, if they chose different connectors -- the
   swap is then the first adaptation) ask: "How do you want to adapt it?"
5. Work through each hole interactively, one at a time. Translate each into
   plain language, ask for a decision only when you genuinely need one, and
   resolve the obvious ones yourself.
6. When done, append a dated entry to "Adaptation history" below (never
   rewrite earlier entries) and commit.

## Holes

The bridge runs out of the box, but these are the gaps an adopter will likely
want to close:

- **Single shared token, all-or-nothing access.** One bearer token grants full
  read of *every* conversation in the workspace and the ability to message any
  agent. There are no per-agent or per-scope tokens. A working replacement is a
  small token registry (token -> allowed agent ids + read/write scope), checked
  in `auth.py`, so you can hand out narrower tokens.
- **Ephemeral public URL.** The public URL is an account-less Cloudflare quick
  tunnel: the hostname is random and changes on every restart, and there is no
  uptime SLA. For a stable, durable URL, swap `tunnel.py` for a named
  Cloudflare tunnel (a Cloudflare account + a reserved hostname) or an ngrok
  reserved domain.
- **The workspace's raw API is unauthenticated on loopback.** The bridge only
  ever exposes *itself* (port 8082, token-gated); the underlying
  `system_interface` API on `127.0.0.1:8000` has no auth of its own. That is
  fine as long as 8000 stays on loopback. But if an adopter separately exposes
  port 8000 publicly (a second tunnel, a reverse proxy), that raw API would be
  wide open -- so don't. Keep only 8082 public.
- **No rate limiting or audit logging on the bridge.** Requests are neither
  throttled nor recorded. A working replacement adds per-token rate limiting
  and an append-only audit log of who messaged/read which agent and when.

## Adaptation history

Each mind that adapts this inspiration appends one dated entry below. Earlier
entries are never rewritten.
