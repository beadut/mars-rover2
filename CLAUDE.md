# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project state

This repository currently contains **no source code**. It holds the Intent and Design (spec) artifacts for a single feature — a Mars Rover command simulator — produced by a custom, skill-driven spec workflow. There is no build, lint, or test tooling yet because implementation (the Build phase) has not started.

When code is eventually added, the target language is **Rust** (fixed by `intent/mars-rover-simulator/intent.md`). At that point, standard Cargo commands (`cargo build`, `cargo test`, `cargo test <name>`, `cargo clippy`, `cargo fmt`) will apply — verify against the actual `Cargo.toml` once one exists rather than assuming.

## Repository content

- `intent/<slug>/intent.md` — the accepted problem statement for a feature: problem, proposed outcome, affected users/systems, constraints, and open questions. Product-decision authority lives here.
- `intent/<slug>/spec.md` — the design spec derived from an accepted intent: numbered requirements (`EX-NN`) each tied back to a phrase in the intent, a scenario (starting state / action / expected result), proposed design, and "réserves" (flagged ambiguities with the Product Owner's decision, author, date, and rationale once resolved).
- `.claude/skills/intent/SKILL.md` and `.claude/skills/spec/SKILL.md` — the two custom skills that generate/revise the files above. Documentation and skill/spec content in this repo is written in **French**; preserve that convention when editing these files.

## The Intent → Design → Build workflow

This repo follows a strict three-phase, human-gated workflow implemented via slash-command skills (`/intent`, `/spec`; a Build-phase skill does not exist yet). The rules below are load-bearing — they are enforced by the skills themselves and must be respected when acting on their behalf:

1. **Intent phase** (`/intent`, in `.claude/skills/intent/SKILL.md`): turns a raw idea into `intent/<slug>/intent.md`. The skill must not invent product decisions, constraints, or technical solutions — anything undecided goes into "Questions ouvertes". It proposes drafts in-conversation and only writes the file after explicit human validation of the draft, on a new branch named `claude/intent-<slug>`. It never commits/pushes/opens a PR without a separate explicit confirmation, and never merges.
2. **Design/Spec phase** (`/spec`, in `.claude/skills/spec/SKILL.md`): reads an *accepted* intent (accepted = merged into `main` via PR decision, not merely present on `main`) and writes `intent/<slug>/spec.md` alongside it. It must not add product constraints beyond the intent, must not write code or a build plan (that belongs to Build), and must record every ambiguity as a "réserve" resolved one at a time with the Product Owner — never decided unilaterally. Every spec.md ends with a "Contexte de génération" section recording the exact triggering prompt and the git commit of each skill version used. Like `/intent`, it stops before saving and asks whether to propose the spec; once accepted it commits, pushes, and opens a PR to `main`, but never merges.
3. **Build phase**: not yet defined in this repo (no skill exists for it). This is where Rust implementation work will happen once a spec is accepted.

Each phase gates the next: don't start Design work on an intent that hasn't been PR-accepted, and don't start Build work ahead of an accepted spec.

## The current feature: Mars Rover simulator

Per `intent/mars-rover-simulator/spec.md`, the accepted design is:
- Rust CLI/library that takes a starting position `(x, y)`, orientation (N/S/E/W), a map with obstacles, and a word-based command list (`Avance`, `Recule`, `Tourne à droite`, `Tourne à gauche`).
- Map input symbols: 🟩/🌳 (forest / tree) and 🟫/🪨 (rocky ground / rock) may coexist on the same map; 🌳 and 🪨 are obstacles, everything else is passable.
- An obstacle or map edge cancels only the deployment command in progress (rover stays in place, edge triggers a warning); subsequent commands in the list still execute normally (decision R-01).
- Map scale: 1 cell = 1 meter, capped at 1000×1000 cells to respect the 1 km² MVP limit (decision R-03).
- Output: the traversed path rendered as ASCII art (distinct character set from the emoji input symbols) — exact character set is an open design proposal, not yet fixed.

## Erreurs récurrentes

Lorsqu’une même erreur se répète deux fois, propose une instruction courte et précise pour l’éviter. Appuie-toi sur les erreurs observées et fais valider cette instruction avant de l’ajouter à CLAUDE.md.

Si une instruction devient obsolète, propose sa correction ou son retrait et attends la validation avant de modifier le fichier.
