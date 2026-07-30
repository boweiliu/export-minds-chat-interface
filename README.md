# export minds chat interface

A REST API (plus a demo web UI) to list, message, and read a minds workspace's agents from outside.

This inspiration puts a single token-gated door in front of a minds workspace's
own chat interface, so you can reach your agents from outside -- a phone, a
script, or another agent running elsewhere. With one bearer token it exposes a
REST + SSE API to list the workspace's agents (with live idle / thinking /
running-a-tool status), send a message straight into any agent's live session,
and read any conversation back (full history plus a live stream). It ships a demo
web console for humans and a zero-dependency Python client for machines, and an
account-less Cloudflare quick tunnel gives it a public URL and QR code -- while
only the token-gated bridge is ever exposed, never the raw API underneath.

This repository is a published **minds inspiration**: a clean, bootable
snapshot of the apps and features a mind built, ready to adapt into your own.
It is NOT the generic workspace template -- it is this specific project.

## Use it

- **Create a new mind from it:** point a new minds workspace at this repo's
  URL. On first boot the mind reads the inspiration and helps you connect your
  own accounts and adapt it.
- **Bring it into an existing mind:** run `/use-inspiration <this repo's URL>`.

## What's inside

- **export minds chat interface** -- [`inspiration-export-minds-chat-interface.md`](inspiration-export-minds-chat-interface.md) (published now)

Each `inspiration-<slug>.md` is the full manifest for that inspiration: what
it is, how it works, the prerequisites it needs, and how to adapt it.
