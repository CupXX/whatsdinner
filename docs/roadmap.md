# What's Dinner — Roadmap and Progress

Status: **Design phase**  
Last updated: 2026-08-20

This file tracks phased work and current progress. It should be updated as decisions are accepted and implementation milestones move forward.

## Current phase

**Phase 0 — Product and architecture definition**

Current objective: turn the existing ChatGPT-based 'What's Dinner' workflow into a reusable DSH Meal Agent with structured state, testable policies, and a path to multi-user use later.

### Completed / agreed

- [x] Repository created and designated as project source of truth.
- [x] Core product goal defined: reduce meal-decision burden rather than ask the user to decide what they want.
- [x] Scope includes both eating out and cooking at home.
- [x] Meal check-in identified as a first-class capability.
- [x] Pantry classified as structured hard state rather than conversational memory.
- [x] Supabase selected as the intended structured state store.
- [x] Memory split conceptually into hard state, long-term memory, short-term/episodic memory, and learned hypotheses.
- [x] Recommendation output direction defined as one strongest recommendation plus a limited number of viable backups.
- [x] Personal intimacy-score gamification removed from the product direction.
- [x] Image input recognized as an important product capability for check-in, grocery selection, and cooking help.
- [x] Future schedule-aware context reserved through a Context Resolver / provider boundary.
- [x] DSH Web UI positioned as a development/testing/inspection surface rather than the final user frontend.
- [x] Final Meal Agent intended to be a narrow custom DSH preset/agent rather than a general-purpose coding agent.

### Still to design before implementation

- [ ] Recommendation Policy.
- [ ] Supabase schema and inventory-ledger invariants.
- [ ] Memory write, expiry, confidence, and promotion rules.
- [ ] Meal check-in workflow and confidence rules.
- [ ] Cooking Session V0 scope.
- [ ] DSH Meal preset composition and exact tool contracts.
- [ ] Legacy Google Sheet migration/import plan.
- [ ] Test scenario suite for agent behavior.

---

## Phase 1 — Agent core and structured state

Goal: produce the first useful Meal Agent that can recommend a meal, read reliable user state, record what was eaten, and maintain pantry state.

Planned outcomes:

- [ ] Initialize the DSH project/runtime structure.
- [ ] Create a narrow custom Meal Agent preset/composition.
- [ ] Create Supabase project/schema for the first supported data domains.
- [ ] Implement persistence/repository layer.
- [ ] Implement initial tools for profile, recent meals, pantry, inventory events, and meal check-in.
- [ ] Implement Recommendation Policy V0.
- [ ] Implement Meal Check-in V0.
- [ ] Import enough existing Google Sheet data to test realistic behavior.
- [ ] Add automated behavioral test cases.
- [ ] Validate manually through DSH Web UI.

### Phase 1 success criteria

A user should be able to say something equivalent to:

> I don't know what to eat tonight.

The Agent should be able to inspect relevant state, make one strong recommendation with limited backups, and explain only the useful reasoning.

After the meal, the user should be able to check in what they actually ate. The system should persist the meal and make any justified pantry changes without relying on unreliable conversational recollection.

---

## Phase 2 — Cooking and shopping workflow

Goal: turn home-cooking support from generic chat into a persistent meal execution workflow.

Planned outcomes:

- [ ] Shopping Session V0.
- [ ] Cooking Session V0 with current-step context.
- [ ] Structured ingredient/quantity handling.
- [ ] Connect planned ingredient usage to pantry ledger events.
- [ ] Support corrections when actual usage differs from the plan.
- [ ] Define safe confidence rules for automatic vs confirmed inventory updates.

Possible later enhancement:

- standardized reusable cooking-flow structures for recipes that worked well for the user.

---

## Phase 3 — Image-assisted interaction

Goal: support the visual interactions already common in the existing workflow.

Planned outcomes:

- [ ] Meal-photo check-in assistance.
- [ ] Supermarket shelf/product comparison.
- [ ] Cooking-state image assistance.
- [ ] Confidence-aware extraction of visible facts.
- [ ] Confirmation flow for uncertain inventory deductions.

Important principle:

Image inference must not silently become hard pantry or profile state when confidence is insufficient.

---

## Phase 4 — Eating-out intelligence

Goal: improve restaurant recommendation quality with current, verifiable local information.

Planned outcomes:

- [ ] Location-aware restaurant candidate retrieval.
- [ ] Opening-hours/current availability checks where supported.
- [ ] Menu/item verification where possible.
- [ ] Restaurant feedback and suppression/weighting rules.
- [ ] Stronger feasibility filtering before ranking.

This phase should replace the old strategy of recommending many options merely to increase the chance that one happens to be usable.

---

## Phase 5 — Schedule-aware meal context

Goal: infer where and when a meal is likely to happen from the user's routine and schedule.

Planned providers may include:

- [ ] University timetable.
- [ ] Google Calendar.
- [ ] Recurring routines.
- [ ] Manual/current location.

All providers should normalize into the same `MealContext` contract so the Recommendation module does not depend directly on calendar/timetable implementation details.

---

## Phase 6 — Consumer product surface

Goal: move beyond DSH Web UI and expose the Agent to ordinary users.

Possible surfaces:

- [ ] Dedicated web application.
- [ ] Account/authentication layer.
- [ ] Pantry UI.
- [ ] Meal history/check-in UI.
- [ ] Recommendation UI.
- [ ] Chat interface for free-form interaction.
- [ ] Optional chat-channel adapters such as WeChat/Telegram if viable.

The consumer frontend should not be required for early Agent development.

---

## Phase 7 — Reusability and open-source packaging

Goal: make the Meal Agent useful beyond the original author's personal deployment.

Planned direction:

- [ ] Keep reusable Agent capability separate from user-specific state.
- [ ] Document tool/data contracts.
- [ ] Provide setup instructions for developers/self-hosters.
- [ ] Decide which parts of the official hosted product, if any, are also open sourced.
- [ ] Support isolated user profiles/state under one hosted deployment.

---

## Working process

The intended development loop is:

```text
Chat/design discussion
        ↓
Update architecture or feature spec
        ↓
Implementation plan
        ↓
Codex implementation + tests
        ↓
DSH Web UI manual inspection/testing
        ↓
Observed issue or new requirement
        ↓
Minimal update to existing spec/architecture
        ↓
Next implementation iteration
```

### Source-of-truth rule

Once a decision is recorded in repository documentation, the repository version is authoritative over old chat wording.

New requirements should normally be handled by:

1. reading the current architecture and relevant spec;
2. identifying the smallest affected boundary;
3. proposing a delta;
4. updating the relevant documentation;
5. implementing against the updated spec.

Large rewrites should happen only when the existing architecture is genuinely no longer fit for purpose.
