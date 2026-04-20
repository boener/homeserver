# 🧠 Home Server

Living documentation and persistent project memory for the `Home Server` ChatGPT project.

This repository is primarily an **internal documentation and memory layer** used to help ChatGPT reason about the system accurately over time.
It exists mainly so the assistant can re-ground itself in the real structure of the server instead of relying on drifting chat context.

## Purpose

This repo is **not** the server itself.
It is the documentation and memory layer around the server.

Use it to track:
- current system state
- architecture and traffic flow
- service-level notes
- major changes over time
- archived historical phases
- non-obvious lessons learned

## Where to Start

- `docs/current-state.md` → canonical snapshot of the system as it exists now
- `docs/architecture.md` → high-level structure, boundaries, and failure domains
- `docs/services/` → service-specific documentation
- `docs/changelog.md` → important changes over time
- `docs/archive/` → superseded phases and preserved historical context

## Operating Model

- Chat is used for active work and troubleshooting
- GitHub is used for durable memory and documentation
- Canonical docs should be updated when meaningful changes occur
- Old or superseded material should be archived rather than left ambiguous
- This repo is maintained primarily to improve future assistance, continuity, and troubleshooting quality

## What Good Documentation Looks Like Here

Good repo content should:
- reflect verified system reality, not guesses
- explain not just what exists, but why it is arranged that way
- help narrow failures into buckets like DNS, routing, proxy, app, or permissions
- preserve important tradeoffs and learning-oriented decisions

## Safety Boundary

This repo is for:
- notes
- documentation
- architecture
- changelogs
- memory

It should **not** be used to modify live server configs or system-critical files unless explicitly requested.
