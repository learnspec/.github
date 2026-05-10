# LearnSpec — Architecture Charter v0.1

> Working document — May 10, 2026  
> Base: LearnMD v0.3 + QuizMD (existing) → extension to the full LearnSpec ecosystem

---

## 1. Overview

LearnSpec is an **open source specification suite** for creating, storing, and exchanging educational content. Every format in the suite is:

- a **valid Markdown file** readable in any editor without specific tooling
- **file-native**: all content lives in flat files — no database, no server, no proprietary tool required to create or read content
- **versionable** via Git (diffable, mergeable)
- **AI-native**: generatable and consumable by LLMs without proprietary APIs
- **interoperable** with other formats in the suite via standardised mechanisms
- **gracefully degradable**: a LearnSpec file must render as readably as possible in any standard Markdown reader, with no prior knowledge of LearnSpec

---

## 2. Suite Formats

| Format | Extension | Role | Status |
|---|---|---|---|
| **TrackMD** | `.track.md` | Sequenced learning paths | To create |
| **LearnMD** | `.learn.md` | Structured educational content | Existing v0.3 — to evolve |
| **QuizMD** | `.quiz.md` | Quizzes, assessments, questionnaires | Existing — to evolve |
| **FlashMD** | `.flash.md` | Flashcards and spaced repetition | To create |
| **DiagramMD** | `.diagram.md` | Diagram syntax specification + reusable diagram blocks | To create |
| **MediaMD** | `.media.md` | Visual resource catalogue with metadata and licences | To create |
| **GlossaryMD** | `.glossary.md` | Definitions and key terms for a corpus | To create |
| **BadgeMD** | `.badge.md` | Micro-credentials linked to a module or quiz | TODO — depends on TrackMD |
| **CertMD** | `.cert.md` | Macro-credentials linked to a full learning track | TODO — depends on TrackMD |

---

## 3. Common Architecture Principles

### 3.1 Graceful Degradation — Founding Principle

**A LearnSpec file must never be unreadable in a standard Markdown environment — only less rich.**

This principle governs every syntax decision in the suite. Whenever a new mechanism is introduced in a spec, the question to ask is: *what does a GitHub, Obsidian, or VS Code reader see if they don't know LearnSpec?*

| Mechanism | Render in a standard reader | Acceptable? |
|---|---|---|
| Fenced block ` ```quiz ` | Raw code block, readable | ✅ |
| Directive `!import ./file.learn.md` | Plain text, visible | ✅ |
| Directive `!ref ./media.media.md` | Plain text, visible | ✅ |
| `![alt](media:slug)` without fallback | Broken image ❌ | ❌ — forbidden |
| `![alt](media:slug "https://fallback.url/img.jpg")` | Image displayed via fallback URL | ✅ |

**Design rule**: any LearnSpec syntax that would produce a broken render (broken image, dead link, unreadable content) in a standard reader is **forbidden without an explicit fallback mechanism**.

**Direct implication for MediaMD**: the symbolic media reference syntax must always include a fallback URL, using Markdown's native `title` field:

```markdown
![Cross-section of the heart](media:heart "https://commons.wikimedia.org/img/heart.svg")
```

A standard reader displays the image via the fallback URL. A LearnSpec player resolves `media:heart` through the referenced MediaMD file and uses the canonical URL with its licence metadata.

---

### 3.2 File-Native — Founding Principle

**All LearnSpec content lives in flat files. No database, no server, no proprietary tool is required to create, read, or version content.**

This principle has concrete implications for every design decision:

- metadata lives in the file's YAML frontmatter, not in an external database
- relationships between files are expressed as relative paths or URLs, not database identifiers
- a complete corpus can be cloned, archived, audited, or migrated by copying a folder
- tooling (player, validator, MCP) is a layer on top of the files, never a prerequisite for reading them

**Git corollary**: file-native makes every LearnSpec corpus naturally versionable, diffable, and collaborative via standard Git tools — with no adaptation required.

---

### 3.3 Universal File Structure

All LearnSpec formats share the same base structure:

```
[Optional YAML frontmatter between --- delimiters]
[Markdown content with format-specific fenced blocks]
```

A file without frontmatter is always valid (Level 0). Richness is progressive.

### 3.4 Level System

Each format defines its own Level 0/1/2 content, but all follow the same logic:

| Level | Principle |
|---|---|
| **0** | Pure Markdown, no tooling required, readable everywhere |
| **1** | YAML frontmatter — metadata, global behaviour |
| **2** | Special fenced blocks and directives — rich content, composition |

**Fundamental rule**: each level is a **strict superset** of the previous one. A Level 0 file is valid at Level 1 and 2.

### 3.5 Frontmatter — Universal Fields

These fields have the **same semantics across all formats** in the suite:

```yaml
---
title: "Content title"            # optional — inferred from first # H1
lang: en                          # REQUIRED — BCP-47 code (en, fr, en-US…)
spec_version: "0.1"               # optional — targeted spec version
author:                           # optional — string or object
  name: Jane Smith
  email: jane@example.com
  url: https://janesmith.com
tags: [python, variables]         # optional — list of free strings
created: 2026-05-10               # optional — ISO 8601 date
updated: 2026-05-10               # optional — ISO 8601 date
license: CC-BY-4.0                # optional — SPDX identifier (e.g. MIT, CC-BY-4.0, CC-BY-SA-4.0)
                                  #            or "custom" for licences not covered by SPDX
---
```

**`lang` is the only universally required field** across all formats.

### 3.6 Format-Specific Fields

Each format may define its own frontmatter fields in addition to the universal fields. These are documented in the relevant format's spec.

### 3.7 File Naming Conventions

| Format | Recommended convention |
|---|---|
| LearnMD | `{slug}.learn.md` |
| QuizMD | `{slug}.quiz.md` |
| FlashMD | `{slug}.flash.md` |
| DiagramMD | `{slug}.diagram.md` |
| MediaMD | `{slug}.media.md` |
| TrackMD | `{slug}.track.md` |
| GlossaryMD | `{slug}.glossary.md` |

The `slug` is lowercase, hyphen-separated, no special characters: `intro-python.learn.md`.

### 3.8 Cross-Format Reference System

#### Shared Philosophy: External Reference Resolution

All LearnSpec directives that accept a file path (`!import`, `!ref`) accept either a local path or an external URL interchangeably. This philosophy rests on three shared principles:

**Player-side URL normalisation** — the spec prescribes no particular URL form. A GitHub `blob`, `raw`, or other link is accepted as-is: it is the player's responsibility to normalise to the fetchable URL. This insulates source files from changes in hosting platforms.

**Optional version pinning** — by default, an external URL points to the main branch of its repository. A commit hash in the URL pins to a precise version, guaranteeing stability over time:

```
!import https://github.com/org/repo/blob/main/module.learn.md        ← follows main
!import https://github.com/org/repo/blob/a3f8c21/module.learn.md     ← pinned
```

**Non-blocking offline behaviour** — if an external resource is unreachable at render time, the player emits a localised warning at the impacted locations and continues rendering. The caching strategy is left to the player implementation.

---

#### Directive `!import`

**Composition** mechanism: includes the content of another LearnSpec file at the current position.

```
!import ./module-2.learn.md
!import ./quiz-module-1.quiz.md
!import https://github.com/org/repo/blob/main/module.learn.md
```

- The extension determines the rendering behaviour
- Imports are recursive (an imported file may itself contain `!import` directives)
- Circular imports are silently skipped

#### Directive `!ref`

**Contextual reference** mechanism: declares a dependency without including content inline. Produces no visible render — establishes a context the player uses in the background (media resolution, glossary term highlighting, etc.).

```
!ref ./anatomy.media.md
!ref https://github.com/neuroneo/commons/blob/main/media/anatomy.media.md
```

#### Directive `!checkpoint`

Marks a named progress point. Available in LearnMD and TrackMD.

```
!checkpoint id:module-1-done label:"Module 1 complete" [type:milestone|read|exercise-complete]
```

- `id`: required, unique within the document
- `label`: optional, text displayed to the learner
- `type`: optional, `milestone` by default

---

## 4. Cross-Format Interoperability

### 4.1 Compatibility Matrix

| Source format | Can import (`!import`) | Can reference (`!ref`) |
|---|---|---|
| **TrackMD** | LearnMD, QuizMD, FlashMD | MediaMD, GlossaryMD |
| **LearnMD** | LearnMD, QuizMD, DiagramMD | MediaMD, GlossaryMD |
| **QuizMD** | DiagramMD | MediaMD, GlossaryMD |
| **FlashMD** | DiagramMD | MediaMD, GlossaryMD |
| **DiagramMD** | — | — |
| **GlossaryMD** | — | — |
| **MediaMD** | — | — |

DiagramMD, GlossaryMD, and MediaMD are **pure leaf formats**: zero dependencies, always consumed, never producers. TrackMD does not import DiagramMD directly — standalone diagrams are embedded in the content formats (LearnMD, QuizMD, FlashMD) it orchestrates.

### 4.2 Media Resolution

When a LearnMD (or any other format) includes an image or visual resource, two mechanisms coexist:

1. **Inline** — standard Markdown link `![alt](url)` pointing to a direct URL
2. **Via MediaMD** — symbolic reference `![alt](media:slug)` resolved from the `.media.md` declared in `!ref`

The `media:slug` mechanism centralises licence and source management in a single MediaMD file without polluting content files.

### 4.3 Glossary Term Resolution

A player compatible with GlossaryMD can automatically highlight and make interactive any terms defined in the referenced glossary, with no modification to the source content.

---

## 5. Validation

### 5.1 Validation Modes

All formats support two modes:

| Mode | Behaviour |
|---|---|
| **Lenient** (default) | Warnings for missing non-critical fields, errors for invalid content |
| **Strict** (`--strict`) | All warnings become errors — recommended for CI/CD pipelines |

### 5.2 Universal Errors (all formats)

| Condition | Level |
|---|---|
| `lang` absent from frontmatter | Warning (Lenient) / Error (Strict) |
| Title absent (no H1 and no `title` in frontmatter) | Warning (Lenient) / Error (Strict) |
| Unclosed fenced block | Error |
| `!import` pointing to a missing file | Warning (Lenient) / Error (Strict) |
| `!checkpoint` missing required `id` attribute | Error |
| Duplicate `!checkpoint` `id` within a document | Error |

Each format additionally defines its own format-specific validation rules.

### 5.3 Spec Versioning

The `spec_version` frontmatter field allows a parser to validate a file against the correct version of the spec. If absent, the parser uses the most recent supported version.

Version format: `MAJOR.MINOR` (e.g. `"0.3"`).

---

## 6. Compatibility and Degradation

The graceful degradation principle is established as a founding principle in **section 3.1**. Concrete behaviours by mechanism:

- Special fenced blocks (` ```quiz`, ` ```flash`, ` ```summary`…) display as raw code blocks — readable but non-interactive
- Directives (`!import`, `!ref`, `!checkpoint`) display as plain text
- YAML frontmatter is displayed or ignored depending on the environment
- `media:slug` references **must** always include a fallback URL (see 3.1)

Each format spec explicitly documents the degraded render of its specific constructs.

---

## 7. AI-Nativeness

All formats are designed to be **generated and consumed by LLMs**:

- Plain-text syntax, no XML or JSON embedded in the body
- Predictable structure: YAML frontmatter then Markdown content
- Fenced blocks named explicitly (`quiz`, `flash`, `card`, `track`…)
- No loops, conditions, or programming logic in the files

**Corollary**: an LLM instructed on LearnSpec specs must be able to generate valid content without intermediate tooling, and an MCP exposing the specs must enable compliant generation.

---

## 8. Architecture Decisions — Resolved Questions

### FlashMD — Richness Levels

Inline LaTeX (`$...$`) is available from **Level 0**, consistent with LearnMD v0.3 where LaTeX is also Level 0. It is a fundamental element of educational content, not an advanced feature.

Visual richness (images, audio) is introduced progressively:

| Level | Allowed content |
|---|---|
| 0 | Markdown text + inline LaTeX |
| 1 | YAML frontmatter + spaced repetition metadata |
| 2 | Images via `media:slug`, rich blocks |

### MediaMD — Licence Management

The `license` field uses **SPDX** identifiers as the canonical reference (e.g. `CC-BY-4.0`, `CC-BY-SA-4.0`, `CC0-1.0`). SPDX covers all Creative Commons and common open source licences.

For cases not covered by SPDX (proprietary licences, rights limited to a private corpus, specific agreements), the value `custom` is reserved. It must be accompanied by a `license_url` or `license_text` field to remain informative.

### TrackMD — Prerequisites

TrackMD v0.1 = ordered linear sequence only. The concept of prerequisites between modules (conditional logic, branching) is deferred to v0.2, once real-world usage feedback is available.

### GlossaryMD — Definition Richness

GlossaryMD definitions allow full **inline Markdown** (bold, italic, links, `code`) as well as **inline LaTeX** (`$...$`). Fenced blocks (Mermaid, quiz, rich examples) are forbidden in definitions — a definition that requires a diagram is a lesson, not a glossary entry.

### Versioning — Version Coexistence

Each file is validated independently against its own declared `spec_version`. The player emits a non-blocking warning if files targeting incompatible versions coexist in the same corpus or TrackMD. If `spec_version` is absent, the parser uses the most recent supported version.

---

## 9. Future Formats — Ideas in Holding

These formats have been identified as relevant but are outside the immediate scope of work. They are documented here to preserve context and decisions made so far.

### TrackMD `.track.md`

Sequenced learning paths: orchestrates LearnMD, QuizMD, and FlashMD content in an ordered progression. Natural reference point for CertMD and BadgeMD.

**Why deferred**: TrackMD is the format that depends on all others. It should be designed once LearnMD, QuizMD, and FlashMD are stable.

**Decisions already made**:
- v0.1 = linear sequence only (no conditional prerequisites)
- Prerequisites between modules deferred to v0.2

---

### BadgeMD `.badge.md`

Micro-credential recognising mastery of a specific skill. Granular, stackable, referenced from a LearnMD or QuizMD. Shares metadata with CertMD (issuer, criteria, expiry, image, verification link) but differs in scope and trigger.

**Why deferred**: depends on TrackMD for its full usage context. To be designed after TrackMD.

**Design direction**: compatibility with the **Open Badges** standard (IMS Global) for interoperability with existing platforms (LinkedIn, Credly, etc.).

---

### CertMD `.cert.md`

Macro-credential attesting mastery of a complete domain. Referenced from a TrackMD, typically requiring a passing score on a formal assessment. Professional or academic scope.

**Why deferred**: depends on TrackMD. To be designed alongside BadgeMD to ensure consistency between the two formats.

**Key distinction from BadgeMD**: CertMD is to BadgeMD what a degree is to a module completion certificate — same family, very different scope and weight. Two separate formats are preferable to a single format with a `type` field, to avoid diluting each spec.

---

### GlossaryMD `.glossary.md`

Definitions and key terms for a corpus. Leaf format referenced via `!ref` from LearnMD, QuizMD, FlashMD. Enables automatic term highlighting and tooltip definitions in a compatible player.

**Why deferred**: less urgent than MediaMD to start. To be designed after MediaMD.

**Decisions already made**:
- Inline Markdown allowed in definitions (bold, italic, links, code, inline LaTeX)
- Fenced blocks forbidden in definitions (Mermaid, quiz, rich examples)

---

### ExerciseMD `.exercise.md`

Open-ended exercises with a problem statement, data, guided steps, and a structured answer key. Distinct from QuizMD (closed assessment) — designed for longer exercises, particularly in maths, physics, and programming. Correction can be AI-assisted.

**Why deferred**: specialised use case, to be evaluated after the core formats are in place.

---

### DiagramMD `.diagram.md`

**Promoted to active format** — decision made during the May 10, 2026 session.

DiagramMD serves a dual role:

**Syntax specification** — formally defines all diagram block types usable across the LearnSpec suite (`mermaid`, `abc`, `chess`, `circuit`, `vega`, `d3`, `smiles`…), their shared attributes (`caption`, `width`, `alt`), and their graceful degradation rules. Other specs (LearnMD, QuizMD, FlashMD) delegate diagram documentation to DiagramMD.

**Standalone file format** — `.diagram.md` files may contain reusable diagrams importable via `!import` from LearnMD, QuizMD, or FlashMD.

DiagramMD is a **pure leaf format**: it imports and references no other LearnSpec format.

---

*This document is the shared working foundation for all LearnSpec specs. It must be updated as decisions are made on each format.*
