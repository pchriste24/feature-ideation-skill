---
name: feature-ideation
description: Generate a ranked list of new feature ideas for an existing codebase by inferring what the product does, imagining how real users use it, finding their friction points, and proposing additions that would make the product meaningfully better. Use this skill whenever the user asks what features to build next, what's missing from their product, what would add value, what they should work on, how to grow the product, what users would want, what pain points exist, what to ship next, or how to differentiate. Trigger on casual phrasings too — "what should I build?", "give me feature ideas", "what would make this better", "ideas for v2", "what's this product missing". Optionally accepts competitor names or links as context, but works without them — the codebase itself is the primary input. This skill is for codebases the user has local access to via Claude Code.
---
 
# Feature Ideation
 
Look at a codebase, figure out what the product *is*, imagine someone actually using it, find the points where that imagined user gets annoyed or stuck or wants more, and propose features that fix those moments. Output a ranked list with reasoning the user can defend in a meeting.
 
This is product thinking, not gap auditing. There's no spec to check against. The job is to invent good ideas that fit the product, then justify them.
 
## Core principle
 
Every feature idea must be **grounded in something concrete** — a code path, a data model, a user flow you can point to in the repo. Generic startup advice ("add analytics", "add a dashboard", "add AI") is worthless unless you can explain *why this product specifically would benefit*, drawing on what's actually there.
 
The strongest ideas come from one of these patterns:
 
1. **Implied next step** — the code clearly does X; the natural follow-up is Y, and Y is missing. (App tracks workouts but never summarizes them. Tool generates reports but can't compare them across time. Library exposes a sync API in a world that wants async.)
2. **User friction in an imagined session** — picture a real person's hands on the product for 10 minutes; what makes them sigh, alt-tab, or give up?
3. **Hidden capability** — the product already has the data or primitives to do something powerful, but doesn't expose it.
4. **Class expectation** — products in this category typically have feature X (search, export, collaboration, undo, keyboard shortcuts, mobile, API, webhooks, theming). Which of those are absent here, and which actually matter for *this* product's users?
5. **Competitor parity or contrast** — when the user names a competitor, what does that competitor do that this product doesn't, and is it worth doing? (Equally important: what does this product do that the competitor doesn't, and could that advantage be sharpened?)
Patterns 1–4 work without any competitor input. Pattern 5 only activates when the user supplies competitor context.
 
## Workflow
 
### 1. Understand the product
 
Before generating any ideas, build a real mental model. Read in this order:
 
- `README.md` and any landing/marketing copy in the repo — what does this product *claim* to be?
- `package.json` / `pyproject.toml` / `Cargo.toml` / equivalent — name, description, dependencies (dependencies tell you a lot: a project depending on `stripe` is a paid product; one depending on `@auth0/nextjs-auth0` has user accounts; one depending on `langchain` is LLM-powered).
- The main entry points — `main.*`, `app.*`, `index.*`, `cli.*`, top-level routes file
- The data models — `models/`, `schema.*`, migrations, type definitions
- The feature surface — list every page, command, endpoint, or public function. This is the user-visible product.
After this pass, write a short internal characterization (you don't have to show the user, but be able to answer):
 
- What does this product do, in one sentence?
- Who uses it? (Solo dev? Team? End consumer? Other developers as a library?)
- How is it used? (Web app? CLI? Library imported into other code? Background service?)
- What is the core loop — the thing a user does over and over?
- What's the product's existing strength — the thing it already does well?
If you genuinely can't tell what the product is from the code, **stop and ask the user**. Generating ideas for a product you've misunderstood produces garbage.
 
### 2. Imagine real usage
 
Pick 2–4 plausible user archetypes based on what you found. Don't invent generic personas — derive them from the code.
 
For a CLI for managing dotfiles: a developer setting up a new laptop; a developer keeping two machines in sync; someone onboarding a junior to their setup.
 
For a project management web app: a team lead with 6 reports running weekly planning; an IC trying to find what they're supposed to do today; an exec scanning status across projects.
 
For each archetype, walk through a session in your head. What do they open the product to do? What are the steps? Where does it get tedious, repetitive, slow, confusing, or impossible? Where do they have to leave the product to get something done?
 
These friction points become idea candidates.
 
### 3. Generate candidates
 
Generate more ideas than you'll ship in the report — aim for 15–25 candidates. Use these prompts to break out of obvious thinking:
 
- **Daily/weekly/monthly views**: what would a user want to do at each cadence that the product doesn't support?
- **First five minutes**: what's missing for someone brand new?
- **Hour 100**: what's missing for a power user?
- **The "I have to leave the app for this" test**: what does the user copy out, paste into another tool, and bring back? Each of those is a feature.
- **Read-side vs. write-side**: products often have one well-developed side. What's underdeveloped on the other?
- **Single-user vs. multi-user**: if it's single-user, would multi-user unlock something? If it's multi-user, are individual workflows neglected?
- **Programmatic access**: is there an API, webhook, CLI, or import/export for the data? Not having one is often a real gap.
- **Recovery and reversibility**: undo, history, soft delete, recycle bin, audit log — many products lack these.
- **Observability for the user**: can the user see what the product just did on their behalf? Logs, history, "what changed".
- **Configurability**: defaults that should be settings; settings that should have presets.
- **Scale-up cliffs**: what breaks or gets painful at 10x current usage (10x more data, 10x more users, 10x more frequency)?
- **Integrations**: with the dependencies already in the repo, what tools is this product's audience clearly using alongside it?
- **AI/automation, but specifically**: not "add AI" — specifically *what task in this product* is currently manual and pattern-based enough that automation would help?
If the user supplied competitor names or links, additionally:
- Mentally enumerate the competitor's notable features. For each, ask: "Would this product's users benefit from a version of that feature, given how this product is positioned?" Skip features that would dilute this product's identity. Borrow features that fit.
- Also ask the inverse: "What does this product do that the competitor doesn't?" Sometimes the best idea is to deepen an existing differentiator rather than achieve parity.
### 4. Filter and rank
 
Not all candidates are worth shipping. Cut anything that:
 
- Doesn't fit the product's identity (great idea, wrong product)
- Solves a problem the imagined users probably don't have
- Is purely cosmetic with no real workflow impact
- Is so vague you can't describe a v1 of it
For the survivors, rate each on two axes:
 
- **Value** (How much does this matter to the imagined users? Does it fix a real friction or unlock real new behavior? 1–5)
- **Effort** (Rough engineering cost given the existing codebase. 1–5, where 1 is small and 5 is large.)
Aim for a final list of **8–12 ideas** with this distribution:
 
- **3–4 quick wins** (high value, low-to-medium effort) — things the user could ship in days or weeks
- **4–6 substantial features** (high value, medium effort) — the meat of the next quarter
- **2–3 ambitious bets** (high value, high effort, possibly differentiating) — bigger moves worth considering
Rank within the list by **value-to-effort ratio**, but include the ambitious bets even if their ratio looks worse on paper — they're there because product strategy isn't only about ROI.
 
## Report structure
 
Output a Markdown file. Save to the repo root as `FEATURE_IDEAS.md` unless the user asks otherwise. Use this exact structure:
 
```markdown
# Feature Ideas
 
**Product:** <one sentence describing what this product is, in your own words>
**Primary users (inferred):** <short list of archetypes>
**Date:** <today>
 
## How these were generated
 
<2–3 sentences: what you read, who you imagined using it, what kinds of friction or opportunity you focused on. If competitor input was provided, name it here.>
 
## Quick wins
 
### 1. <Imperative title — e.g., "Add `--watch` mode to the build CLI">
 
- **Value:** ★★★★☆  **Effort:** ★★☆☆☆
- **Why this matters:** <2–3 sentences. Anchor in a specific user moment or code observation. Not generic.>
- **What it looks like:** <1–3 sentences describing v1. Concrete enough to be a ticket.>
- **Code anchor:** <file or area of the repo where this would land, or which existing feature it builds on>
 
### 2. ...
 
## Substantial features
 
### N. ...
(same fields)
 
## Ambitious bets
 
### M. ...
(same fields, plus:)
- **Why it's ambitious:** <what makes this a bigger swing — new subsystem, new audience, technical risk>
 
## Ideas considered but cut
 
<Optional. Short list of candidates you generated but didn't include in the main list, with one-line reasons. Helps the user trust the process and surfaces optionality. Skip this section if the cuts were obvious or there's nothing interesting to say.>
```
 
Keep entries tight. The goal is a list the user can scan in three minutes and pick from, not an essay.
 
## What not to do
 
- **Don't suggest generic features without product-specific reasoning.** "Add dark mode" isn't an idea unless you can say why it matters here. ("Users run this CLI's TUI dashboard in tmux for hours; eyestrain is a real complaint pattern for terminal UIs at this density of text" — now it's an idea.)
- **Don't propose features that contradict the product's identity.** A minimal Unix-philosophy CLI doesn't need a GUI. A privacy-focused tool doesn't need cloud sync as its headline feature.
- **Don't pad the list.** 8–12 strong ideas beat 25 mediocre ones. If you only have 6 good ideas, ship 6.
- **Don't conflate refactors with features.** "Refactor the auth module" is not a feature idea. (It might be a good thing to do — separate conversation.)
- **Don't lean on competitor parity if no competitor was named.** Without that input, ground every idea in the product itself.
- **Don't hallucinate user research.** You don't know what users said in interviews — you imagined a session. Phrase imagined friction as imagined ("a user opening this for the second day in a row would likely…"), not as reported fact.
## Worked example
 
Imagine a small CLI tool, `notesync`, that syncs a folder of Markdown notes to a remote git repo. README says: *"`notesync` keeps your notes folder in sync with a private git remote. Run `notesync push` to upload, `notesync pull` to download. Conflicts are written to `.notesync/conflicts/`."*
 
After reading the code: there's a `push`, `pull`, and `init` command. Conflicts are dumped as `.orig`/`.theirs` files. No daemon, no scheduling, no inspection commands. Single-machine focused. Dependencies: `gitpython`, `click`, `watchdog` (interesting — already imported but apparently unused).
 
Imagined users: someone with notes on laptop + desktop syncing between them; someone on a single machine using it as backup; a writer who edits across phone and laptop.
 
Friction points (selected):
- After a sync conflict, the user has to dig into `.notesync/conflicts/` and hand-resolve files. There's no inspection or interactive resolution.
- `watchdog` is a dependency but there's no `notesync watch` command — so users have to remember to run `push` manually.
- No way to see "what changed in the last sync" — useful for someone who edits on a phone and wants to confirm changes landed.
- No filtering — you sync everything in the folder. Someone with a `drafts/` subfolder they want to exclude has no recourse.
Resulting top entries (abbreviated):
 
```markdown
## Quick wins
 
### 1. `notesync watch` — auto-sync on file change
 
- **Value:** ★★★★★  **Effort:** ★★☆☆☆
- **Why this matters:** The cost of forgetting to run `push` is silent data divergence — the exact failure mode users adopted a sync tool to avoid. The repo already depends on `watchdog`, suggesting this was intended.
- **What it looks like:** A `notesync watch` command that observes the notes folder and runs `push` (debounced, e.g. 30s after last change) until the user stops it. Logs each sync to stdout.
- **Code anchor:** New command in `cli.py`; reuse existing `push` logic; `watchdog` already in `requirements.txt`.
 
### 2. `notesync status` — show local vs. remote diff
 
- **Value:** ★★★★☆  **Effort:** ★★☆☆☆
- **Why this matters:** Users currently have no way to know whether they're in sync without running a `push`/`pull` and reading git output. A status command turns "is my work safe?" from a guess into a check.
- **What it looks like:** Lists files changed locally since last sync, files changed remotely since last sync, and any unresolved conflicts. Read-only.
- **Code anchor:** New command in `cli.py`; uses the same `gitpython` repo handle as `push`/`pull`.
 
## Substantial features
 
### 3. Interactive conflict resolution
 
- **Value:** ★★★★★  **Effort:** ★★★☆☆
- **Why this matters:** The current behavior — dropping `.orig` and `.theirs` files in a hidden folder — is the kind of moment that drives users to abandon a tool. Most conflicts in note-sync are small (paragraphs added on each side); a TUI walking the user through them turns a 20-minute frustration into a 2-minute task.
- **What it looks like:** `notesync resolve` opens each conflicted file with a side-by-side TUI (or invokes `$EDITOR` with merge markers if no TUI). User picks left/right/both/edit per hunk. Saves resolved file and removes from `.notesync/conflicts/`.
- **Code anchor:** New module; conflict files already isolated in `.notesync/conflicts/`.
 
## Ambitious bets
 
### 8. Mobile companion via remote-first architecture
 
- **Value:** ★★★★★  **Effort:** ★★★★★
- **Why it's ambitious:** Requires a hosted component or a clever delegation model; expands the product from "CLI sync tool" to "notes platform". Worth considering only if the user wants to grow audience beyond CLI-comfortable users.
- **Why this matters:** A meaningful share of plain-text-notes users edit on phones. Right now those users either leave `notesync` for Obsidian/Bear or run hacky shortcuts.
- **What it looks like:** A web reader (initially read-only) hosted by the user that authenticates against the same remote, rendering Markdown with backlinks. Phase 2: web write support, phase 3: mobile app.
- **Code anchor:** Greenfield; reuses git remote as backend.
```
 
Notice each entry: specific to *this* product, anchored in *this* code, derived from imagined user moments. That's the bar.
 
## When you can't infer enough
 
If after reading the repo you genuinely can't tell what the product does, who uses it, or how — don't proceed. Ask the user 2–3 specific questions instead. Examples: "I see this is a Python library exposing a few classes around scheduling, but I can't tell whether the audience is application developers embedding it or ops folks running it as a service. Which is it?"
 
A short clarifying turn at the start beats a useless ideas list at the end.