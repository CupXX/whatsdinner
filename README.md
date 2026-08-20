# What's Dinner

An ADHD-friendly meal decision and cooking agent built around DeepSeek Harness (DSH).

The core goal is simple: when the user does not want to spend mental effort deciding what to eat, the agent should reduce the decision burden, make a strong recommendation, and learn from what actually happened afterward.

## Current status

This repository is in the **architecture and product-design phase**. No production implementation has started yet.

The current design direction includes:

- one clear best recommendation plus a small number of viable backups;
- support for both eating out and cooking at home;
- meal check-in as a first-class workflow;
- accurate pantry state stored as structured data rather than conversational memory;
- short-term, long-term, and inferred preference memory with different confidence rules;
- Supabase as the intended structured state store;
- DSH as the agent runtime / harness;
- DSH Web UI as the primary development and runtime inspection interface during early development;
- future support for schedule-aware meal context, such as university timetables and calendar-derived meal locations;
- image input as an important future interaction mode for meal check-ins, grocery selection, and cooking assistance.

## Documentation

The repository documentation is intended to be the project's source of truth. Chat discussions are exploratory; once a decision is accepted, it should be reflected here.

Planned documentation structure:

- `docs/architecture.md` — system-level architecture and boundaries
- `docs/roadmap.md` — phased implementation roadmap and current progress
- `docs/decisions/` — architecture decision records when decisions need durable rationale
- `docs/specs/` — detailed feature specifications after each subsystem is designed

## Design principle

**Shared agent capability, isolated user state.**

The agent logic should be reusable, while each user's meals, pantry, preferences, memories, and future schedule context remain isolated by user identity.
