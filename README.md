# 🧠 Home Server

Living documentation and persistent project memory for the `Home Server` ChatGPT project.

This repository is primarily used to keep the system state, architecture, service roles, and important lessons current over time.

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
- `docs/architecture.md` → high-level structure and request flow
- `docs/services/` → service-specific documentation
- `docs/changelog.md` → important changes over time
- `docs/archive/` → superseded phases and preserved historical context

## Operating Model

- Chat is used for active work and troubleshooting
- GitHub is used for durable memory and documentation
- Canonical docs should be updated when meaningful changes occur
- Old or superseded material should be archived rather than left ambiguous

## Safety Boundary

This repo is for:
- notes
- documentation
- architecture
- changelogs
- memory

It should **not** be used to modify live server configs or system-critical files unless explicitly requested.
