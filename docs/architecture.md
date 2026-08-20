# What's Dinner — Architecture v0

Status: **Working architecture / source of truth**  
Last updated: 2026-08-20

This document records decisions that have been agreed during product design. Items that are still open are explicitly marked as pending rather than silently treated as final.

## 1. Product goal

What's Dinner is an ADHD-friendly meal decision agent.

Its primary job is **not** to ask the user what they feel like eating. Its value appears when the user does not want to spend mental effort making that decision.

The agent should:

1. understand the context of the upcoming meal;
2. reduce the candidate space using hard constraints and learned preferences;
3. present **one strongest recommendation** plus a small number of genuinely viable backups;
4. support either eating out or cooking at home;
5. help with shopping and cooking when the chosen meal is home-cooked;
6. capture what the user actually ate through a meal check-in;
7. use that outcome to update meal history, pantry state, feedback, and appropriate memories;
8. improve future recommendations without inventing facts about the user.

The product should reduce decision load rather than move the choice back to the user through repeated preference questions.

## 2. Core loop

```text
Resolve meal context
        ↓
Generate and filter candidates
        ↓
Rank candidates
        ↓
1 best recommendation + limited backups
        ↓
      chosen path
      ↙        ↘
 eat out      cook at home
                 ↓
          shopping (optional)
                 ↓
          cooking session
                 ↓
              check-in
                 ↓
      meal history / pantry / memory
                 ↓
      affects future recommendations
```

Meal recommendation and meal check-in are both first-class product capabilities.

## 3. Runtime and platform boundaries

### DSH

DeepSeek Harness (DSH) is the intended agent runtime / harness.

DSH is responsible for:

- agent orchestration;
- tool invocation;
- session/runtime behavior;
- skills and policy composition;
- runtime observability during development.

The production Meal Agent should be a narrow custom agent/preset rather than a general-purpose coding agent with unnecessary shell, Git, code-editing, or subagent capabilities.

### DSH Web UI

During early development, DSH Web UI is the main manual testing and runtime-inspection environment.

It is a developer surface, not the final consumer product UI.

The final user-facing interface may later be a dedicated web application and/or chat-channel adapter.

### Product frontend

A dedicated consumer frontend is intentionally out of scope for the first agent-engineering milestone.

The agent should remain headless enough that multiple clients could eventually use the same core runtime:

```text
Web app ─────┐
Chat channel ├──→ Meal Agent → user-specific state
Other client ┘
```

## 4. Shared capability, isolated user state

The Meal Agent's reasoning logic, tools, policies, and skills should be reusable across users.

Each user's data must remain isolated by `user_id`, including:

- meal history;
- pantry;
- preferences;
- memories;
- restaurant feedback;
- recommendations and their outcomes;
- future schedule / location context.

A single hosted application should be able to serve many users without creating a separate deployment per user.

## 5. Major modules

### 5.1 Context Resolver

Purpose: determine the context in which the next meal is likely to happen.

Target normalized output:

```text
MealContext
- date/time
- meal slot
- expected region/location
- at_home
- available time window
- source
- confidence
```

The Recommendation module should depend on `MealContext`, not directly on a university timetable, Google Calendar, GPS provider, or any specific schedule source.

#### V0

Use simple explicit/default context only.

#### Future extension

Support context providers such as:

- university timetable;
- Google Calendar;
- work schedule;
- recurring routine;
- current/manual location;
- other schedule sources.

This is deliberately designed as an extension point, but schedule import is **not a V0 requirement**.

### 5.2 Recommendation

Purpose: choose a meal when the user does not want to make the decision themselves.

Confirmed product behavior:

- default to making a recommendation rather than asking broad preference questions;
- filter out non-viable options before ranking;
- return one clear strongest recommendation;
- return only a small number of useful backups;
- do not force a fixed number of recommendations if only one or two candidates are genuinely good;
- consider recent meal history to avoid repetitive recommendations;
- consider pantry state for home-cooking recommendations;
- consider user preference and prior feedback;
- do not fabricate restaurants, menu items, availability, or user facts.

The detailed Recommendation Policy is still pending and will receive its own feature spec.

### 5.3 Meal Check-in

Purpose: record what actually happened after the recommendation or meal.

A check-in should be able to capture:

- what was eaten;
- meal type/category;
- restaurant/source or home-cooked source;
- whether a recommendation was accepted, rejected, or changed;
- rating / explicit feedback when available;
- notes and image-derived observations when confidence is sufficient;
- pantry changes when a home-cooked meal consumed tracked ingredients.

Check-in is a key input to future recommendation quality and pantry accuracy.

### 5.4 Pantry

Pantry is **hard structured state**, not conversational memory.

The agent must not answer inventory questions from vague recollection when structured state is available.

Preferred model: an inventory ledger.

Example event types:

```text
PURCHASE
CONSUME
DISCARD
EXPIRE
CORRECTION
```

This allows current stock to be traced to concrete state changes rather than overwritten guesses.

High-confidence pantry changes may be recorded automatically. Low-confidence inferred consumption should be confirmed rather than silently mutating inventory.

### 5.5 Shopping Session

Purpose: help select and acquire ingredients for a planned meal.

Important boundary:

**Recommendation to buy an item is not evidence that it was purchased.**

Inventory should only change after a purchase is confirmed through an explicit user statement or another sufficiently reliable signal.

Detailed workflow is a later milestone.

### 5.6 Cooking Session

Purpose: retain the state of an active cooking task rather than treating every cooking question as a fresh chat.

Future structured session data may include:

- dish;
- servings;
- planned ingredients and quantities;
- ordered cooking steps;
- current step;
- deviations/problems;
- actual ingredient usage.

This enables context-aware help such as responding to a cooking problem without asking the user to repeat what they are making.

The first implementation can begin simpler, but the architecture should not prevent structured cooking flows later.

## 6. State and memory model

Not everything the agent needs to know is the same kind of memory.

### 6.1 Hard State

Must be stored as structured facts, intended for Supabase.

Examples:

- pantry quantities;
- meal history;
- recommendation outcomes;
- meal check-ins;
- schedule facts;
- active cooking/shopping session state;
- location context when sourced from structured providers.

### 6.2 Long-term Memory

Durable behavioral preferences or stable user habits.

Examples:

- dislikes unnecessary onion chopping;
- usually eats alone;
- prefers certain reliable restaurants/dishes;
- recurring cooking preferences.

Long-term memory should not be created from weak evidence.

### 6.3 Short-term / Episodic Memory

Temporary context with an expiry or limited relevance window.

Examples:

- wants lighter food this week;
- does not want fried food today;
- current cooking state;
- temporary dietary mood or constraint.

Temporary state must not silently become permanent profile data.

### 6.4 Learned Hypothesis

Inferred preference that the user has not explicitly confirmed.

A hypothesis should preserve:

- the candidate inference;
- confidence;
- supporting evidence;
- contradictory evidence when relevant.

Repeated evidence can strengthen a hypothesis. A weak inference must not immediately become a hard user preference.

## 7. Supabase direction

Supabase is the intended structured state store from an early milestone rather than a long-term Google Sheets/JSON backend.

The existing Google Sheet is treated as a useful migration/source dataset, not the desired final persistence model.

Likely domains include:

```text
users
user_preferences
memories
meal_history / meal_checkins
recommendations
inventory_items
inventory_events
restaurant_feedback
cooking_sessions
shopping_sessions
meal_context
```

Exact schema names and relationships are **not yet final** and should be designed in a dedicated storage/schema spec rather than inferred from this list.

The Agent should access persistence through narrow repository/tool interfaces rather than embedding raw database logic throughout prompts or agent code.

## 8. Image input

Image input is considered a first-class future product capability, even if the earliest implementation milestone remains text-first.

Important use cases:

- meal check-in photos;
- supermarket shelf/product selection;
- cooking-state/problem photos.

Image-derived facts must be confidence-aware. The system should distinguish what is visible from what is merely inferred.

## 9. Recommendation outcome signal

The previous personal 'intimacy score' is not part of this architecture.

However, recommendation outcomes remain important training/learning signals:

```text
recommended
→ accepted / rejected / changed
→ actual meal
→ explicit feedback / rating
```

These outcomes should improve future ranking and user understanding without turning them into an arbitrary gamification score.

## 10. Initial tool surface

The exact tool set is still subject to feature-spec refinement, but the initial narrow surface is expected to include capabilities equivalent to:

```text
get_user_profile
get_recent_meals
get_inventory
update_inventory / record_inventory_event
log_meal / create_checkin
```

Additional tools should be added only when a real workflow requires them.

## 11. Explicit non-goals for the first milestone

The first implementation should not attempt all of the following at once:

- final consumer web application;
- WeChat / Telegram / other chat-channel integration;
- automatic university timetable import;
- Google Calendar integration;
- complex multi-agent architecture;
- general-purpose shell/code-editing permissions for the Meal Agent;
- autonomous purchase execution;
- fully automated computer vision inventory deduction from arbitrary photos.

## 12. Open design questions

The following require dedicated design before implementation:

1. Recommendation Policy: hard filters, ranking factors, mandatory reads, and allowed clarifying questions.
2. Exact Supabase schema and ledger invariants.
3. Memory write/promotion/expiry rules.
4. Check-in confidence rules and image-assisted check-in workflow.
5. Cooking Session V0 scope.
6. DSH custom Meal preset composition and tool registration.
7. Migration strategy for the existing Google Sheet history.

These should be resolved through separate feature specs rather than by repeatedly rewriting the entire architecture.
