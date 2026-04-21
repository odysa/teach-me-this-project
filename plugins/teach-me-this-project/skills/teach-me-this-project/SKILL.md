---
name: teach-me-this-project
description: "Generate a publish-ready interactive HTML tutorial that explains how any software project works — from first principles to implementation details. The output is a single-file interactive website with animated visualizations, suitable for GitHub Pages. Use when asked to create a tutorial, explainer, or interactive docs for a codebase."
when_to_use: "generate tutorial, create tutorial, explain this codebase, interactive docs, how does this project work, teach me this codebase"
argument-hint: "[project_path] [audience: beginner|intermediate|expert] [--design brand] [focus area]"
effort: max
---

# Interactive Project Tutorial Generator

Generate a comprehensive, publish-ready interactive HTML tutorial that explains how any software project works — from first principles to implementation details. The output is a single-file interactive website with animated visualizations.

## Parse Arguments

Extract from `$ARGUMENTS`:

- **PROJECT_PATH**: Path to codebase or GitHub URL (required). If omitted, use the current working directory.
- **AUDIENCE**: `beginner`, `intermediate` (default), or `expert`
- **DESIGN**: Optional design system brand (default: `claude`). Use `--design <brand>` to apply a different brand. Run `npx -y getdesign@latest add <brand>` to install, then read the resulting `DESIGN.md` for tokens. If the brand is `claude` or omitted, use the built-in Claude design tokens below.
- **FOCUS**: Optional specific subsystem to emphasize (e.g., "the scheduler", "the plugin system")
- **OUTPUT**: Output filename. Default: `<project_name>_tutorial.html`, where `<project_name>` is the last path segment of `PROJECT_PATH` (or the repo name if it's a GitHub URL), lowercased with non-alphanumerics replaced by `_`. Examples: `./mini-sglang` → `mini_sglang_tutorial.html`; `https://github.com/vllm-project/vllm` → `vllm_tutorial.html`.

If the project is a GitHub URL and not cloned locally, clone it to `/tmp/<repo-name>` first.

### Design System Resolution

1. If `--design claude` or no `--design` flag: use the **Built-in Claude Design Tokens** in Appendix A below.
2. If `--design <other-brand>`: run `npx -y getdesign@latest add <brand>` in `/tmp`, read the resulting `DESIGN.md`, and extract color palette, typography, spacing, shadows, border-radius, and do's/don'ts. Apply those tokens instead of the Claude defaults in Phase 4.

---

## Operating Principles

These principles govern how the skill itself is authored and how an executor should interpret it. Read them before the step-by-step.

1. **Encode invariants, not procedures.** Say what must be true of the output, not what steps to follow. Invariants are checkable; procedures force judgment calls.
   - ❌ "Include a `prefers-color-scheme: dark` override."
   - ✅ "No color outside the active design system's palette may appear in any emitted CSS, `<style>` block, or SVG attribute."

2. **Gate generic patterns on brand capability.** Default templates assume features not every brand has. Before emitting a pattern (dark mode, gradient, neumorphic shadow, glass blur), verify the design system supports it. If it doesn't: omit the pattern entirely. Never invent tokens.

3. **Prefer one correct reference over three paragraphs of prose.** Executors pattern-match. Give them the exact pattern you want reproduced.

4. **Every bug found once goes in Known Traps.** Otherwise every run repeats it. Treat this skill like code — it has a changelog.

## Known Traps

Non-obvious failure modes from past runs. Every executor must read this list and avoid each pitfall — don't rediscover these on your own run.

- **SVG presentation attribute override.** CSS class rules (`svg.diagram .node { fill: ... }`) **override** inline `fill="..."` on the element. A `<rect class="node" fill="#fff">` will render with the class's fill, not `#fff`. Mitigation: author SVG CSS with `:where(.class)` (zero specificity) so inline attributes win when explicitly set. The scaffold in Step 4.2 uses `:where()` throughout for this reason.
- **`<text>` inside an accent `<rect>` inherits light text on a dark background.** When a node is filled with `var(--accent)`, its child `<text>` must be switched to an inverse color (`var(--bg-secondary)` or `#faf9f5`) or it becomes unreadable dark-on-dark. Solution: always set the text color explicitly on accent nodes, or wrap the accent node + label in a group with a class that cascades the right text color.
- **Mixing diagram container backgrounds with card backgrounds.** A figure inside a chapter should share the chapter's surface — never a dark container inside a light chapter (or vice versa). If a chapter sits on `var(--bg-secondary)`, every SVG `<rect class="lane">` inside it should also use `var(--bg-secondary)`, not the raw page `--bg`.
- **Generic dark-mode media query on light-only brands.** Claude, and many editorial/paper brands, define only a light palette. Emitting `@media (prefers-color-scheme: dark) { :root { --bg: #141413; ... } }` forces the executor to invent dark tokens that aren't in DESIGN.md — the page then swaps to an un-branded palette whenever the user's OS is in dark mode. Only emit the override when DESIGN.md has an explicit dark variant.
- **Line numbers drift between releases.** `file.py:L42-L78` today ≠ `file.py:L42-L78` tomorrow. Step 1.5 must re-verify every citation; never trust a line range you haven't grep'd this session.
- **`Math.random()` in demos.** Two users comparing notes must see the same numbers. Every demo uses the seeded `rand()` helper from Step 4.2; resets re-seed from a fixed constant.
- **Writer drift into prose during the raw pass.** Step 2.2 is bullets only. Prose at the raw stage forces a fact/prose entangled review, which defeats the two-pass design. Step 2.3 reviewers must kick prose back to bullet form.
- **`<foreignObject>`, Mermaid, base64 PNG diagrams.** All break design-token inheritance. Diagrams must be authored as pure SVG with classes from the scaffold.
- **Overlapping labels and boxes in SVG diagrams.** LLM-authored SVG can't measure rendered text width — agents guess from character count and routinely emit boxes that clip their own labels, or labels drawn on top of edge lines. The fix is a small set of sizing rules enforced at authoring time; see **Diagram Layout Discipline** in Step 4.2. Step 4.3 gate greps node width against the longest contained label text — boxes too narrow fail the gate.
- **Unlinked paper references.** Writers drop the citation text but forget the URL, shipping "Fast Inference from Transformers via Speculative Decoding (Leviathan et al., 2023)" with no `<a href>` around it — the reader can see the paper name but can't click to read it. Every entry in `.further-reading` must wrap an `<a href="https://…">`; bare text citations fail the Step 4.3 gate. If no URL exists for a paper, don't cite it.
- **Dark code blocks on a light-brand page.** A common default is "code is always dark, regardless of theme" — wrong for Claude and other light-parchment brands. Code blocks must use the brand's `--code-bg` token. On Claude that's `#f0eee6` (warm cream), not `#141413`. Syntax-highlight colors must be contrast-tested against the actual code surface, not copied from a dark-mode preset.
- **Page-level scroll instead of content-level scroll.** If `.content` (the chapter column) has no bounded height, the entire browser window scrolls as one long strip — the sidebar scrolls away, the active chapter heading scrolls away, and switching chapters drops you wherever you happened to stop. The fix is structural CSS, not chapter length: the chapter container takes `height: 100vh; overflow-y: auto;` so scrolling happens **inside** the content column. Sidebar stays fixed, chapter scroll resets on navigation, page never extends past the viewport.

When a new bug is found during a run, add it here — the skill has a changelog.

## Agent Dispatch Table

This skill is orchestration-heavy. The orchestrator (main loop) holds the plan; subagents do the bounded, parallelizable work. Every step below that says "dispatch N subagents" **must** use the Agent tool — never inline the work into the orchestrator's context.

| Step | Work | Dispatch |
|------|------|----------|
| 1.1 Architecture Discovery | Survey codebase, find papers | 1 subagent (`Explore`, thoroughness: very thorough) |
| 1.2 Concept Dependency Graph | Synthesize survey into graph | Orchestrator (stateful synthesis) |
| 1.3 Deep Dives | Per-concept research, source extraction | **3 parallel subagents** (`Explore`), split by layer (foundations / orchestration / execution) |
| 1.4 Chapter Planning | Read all Phase 1 research, decide final chapter list (merges/splits/drops/adds), produce binding `chapter_plan.md` | 1 subagent (`general-purpose`) |
| 1.5 Citation Gate | Verify every `file:LN-LN` tag | **3 parallel subagents** (`general-purpose`), same layer split |
| 1.6 Consistency Bible | Synthesize terminology / notation / abbreviations / cross-ref map | 1 subagent (`general-purpose`) |
| 2.2 Raw Draft | Write structured bullets/facts/snippets per chapter | **3 parallel subagents** (`general-purpose`), same layer split |
| 2.3 Raw Review | Fact/citation/consistency review on bullets (cheap) | **N parallel subagents** (`general-purpose`), one per raw chapter file; fires as each writer completes |
| 2.4 Prose Pass | Turn approved raw content into polished paragraphs | **3 parallel subagents** (`general-purpose`), same layer split |
| 2.5 Prose Polish | Flow, tone, callout placement — facts already vetted | **N parallel subagents** (`general-purpose`), one per prose chapter file |
| 3 Round 1 Cross-Chapter Consistency | Terminology drift, redundancy, cross-refs across chapters | **2 parallel subagents** (`general-purpose`), split by chapter halves |
| 3 Round 2 Pedagogy | Role-play target audience | 1 subagent (`general-purpose`), fresh context |
| 4.0 Stage 1 Shell | Write CSS/JS skeleton | 1 subagent (`frontend-developer`) |
| 4.0 Stage 2 Assembly | Stitch chapters into shell | Orchestrator (sequential Edit inserts) |
| 4.2 Interactive Demos | Per-chapter demo HTML + JS | **3 parallel subagents** (`general-purpose`), same layer split |
| 4.3 Visual Consistency Gate | Grep-level invariant checks on assembled HTML | 1 subagent (`general-purpose`) |

**Why this matters**: the orchestrator's context is the scarce resource. Anything the orchestrator reads directly — source files, full drafts, demo JS — crowds out the plan. Subagents return only short summaries (file lists, counts, diffs). The orchestrator stays slim and survives the whole run.

**Return-value discipline**: every subagent's final message must be ≤200 words — a list of files written, a summary of issues found and fixed, or a count. Never quote chapter text, code blocks, or full logs back to the orchestrator. If an agent needs to pass detail forward, it writes to a file and reports the path.

---

## Phase 1: Deep Codebase Investigation

### Step 1.1: Architecture Discovery

Dispatch **1 subagent** (type: `Explore`, thoroughness: "very thorough") to survey the codebase:

- Map the directory structure and identify major modules
- Read README, ARCHITECTURE.md, CONTRIBUTING.md, and any design docs
- Identify entry points (main, CLI, server startup)
- Trace the primary data flow end-to-end (e.g., request → response, input → output)
- List key abstractions: core data structures, interfaces, protocols
- Identify external dependencies and what role they play
- **Find associated papers and references**: search README, docs/, comments, and `CITATION.cff` for arXiv links, DOIs, paper titles, or blog posts. Many open-source projects are implementations of research papers — finding these is critical for the tutorial's credibility and depth. Also web-search for `"<project-name>" paper site:arxiv.org` to find papers the repo doesn't link directly.

### Step 1.2: Concept Dependency Graph

From the architecture survey, build a **concept dependency graph** — the order in which concepts must be taught so each builds on the previous:

```
Example for a web framework:
  HTTP basics → Routing → Middleware → Request/Response → ORM → Templates → Auth
Example for an ML system:
  Tensors → GPU → Model → Attention → KV Cache → Batching → Scheduling
```

This graph determines chapter order. Target: **8-16 chapters**.

### Step 1.3: Deep Dives per Concept

Dispatch **3 parallel subagents** (type: `Explore`) to investigate the codebase. **Split by architectural layer, not numeric thirds** — concepts in the same layer share vocabulary and code paths; concepts across layers don't. A thirds-split forces agents to research each other's dependencies in parallel, producing inconsistent terminology and duplicated work.

Group the concepts from the dependency graph into these buckets (adjust names per project domain):

- **Agent A — Foundations**: primitives, data structures, memory model, core abstractions. Covers anything the other layers import from.
- **Agent B — Orchestration**: schedulers, routers, state machines, control flow. The "manager" layer that decides what runs when.
- **Agent C — Execution**: hot paths, kernels, forward passes, the actual work. The layer that consumes foundations and obeys orchestration.

If a concept straddles two buckets, assign it to the **lower** layer (it's a dependency, not a consumer). If a bucket is empty for a given project, redistribute — three agents by layer is the ceiling, not a quota.

Each agent produces for every concept:

```markdown
## Concept: <Name>

### What It Does
One-paragraph summary a newcomer can understand.

### Why It Exists
The problem it solves. What goes wrong without it.

### How It Works
Step-by-step explanation with code references (file:line).
Key data structures and their fields.
Key algorithms and their complexity.

### Source Code Snippets
For each core mechanism, extract the **actual source code** (not pseudocode):
- 10-40 lines (essential logic, not boilerplate)
- Trimmed of logging, error handling, irrelevant branches
- Annotated with inline `# ←` comments on non-obvious lines
- Tagged with exact file path and line range: `# source: path/to/file.py:L42-L78`

Include at minimum:
- The primary data structure definition (class/struct with key fields)
- The core algorithm (the "hot loop" or main dispatch logic)
- One non-obvious helper that reveals a design decision

### Key Design Decisions
Why this approach over alternatives. Trade-offs made.

### Papers & References
For each concept, find the original paper or authoritative source that introduced
the technique. Many codebase techniques originate from research papers — the
tutorial should cite them so readers can go deeper.

Search strategy:
1. Check code comments and docstrings for paper citations, arXiv links, or "Based on..."
2. Check the project's README/docs for a references section
3. Web-search: `"<technique name>" paper site:arxiv.org` (e.g., "speculative decoding paper site:arxiv.org")
4. Web-search: `"<technique name>" <project name>` for blog posts explaining the technique

For each paper found, record:
- Title, authors, year
- arXiv or DOI link
- One-sentence summary of what the paper contributes
- Which codebase files implement the paper's ideas

Example:
```
- **Speculative Decoding**: "Fast Inference from Transformers via Speculative Decoding"
  (Leviathan et al., 2023). arXiv:2211.17192. Introduced the draft-then-verify paradigm.
  Implemented in: speculative/eagle_info.py, speculative/dflash_info.py

- **RadixAttention**: "SGLang: Efficient Execution of Structured Language Model Programs"
  (Zheng et al., 2024). arXiv:2312.07104. Introduced radix tree KV cache for prefix sharing.
  Implemented in: mem_cache/radix_cache.py

- **Flash Attention**: "FlashAttention: Fast and Memory-Efficient Exact Attention"
  (Dao et al., 2022). arXiv:2205.14135. IO-aware attention algorithm.
  Implemented in: layers/attention/flashinfer_backend.py

- **Continuous Batching**: "Orca: A Distributed Serving System for Transformer-Based
  Generative Models" (Yu et al., 2022). arXiv:2206.01698. Introduced iteration-level scheduling.
  Implemented in: managers/scheduler.py
```

### Interactive Demo Idea
A specific visualization: what the user inputs/clicks, what animates, what insight it delivers.
```

### Step 1.4: Chapter Planning (lock in the structure before drafting)

The initial dependency graph from Step 1.2 was built off the architecture survey alone — shallow. After Step 1.3's deep dives the picture is clearer: some concepts should merge, some should split, some turn out to be too thin for their own chapter, some were missing entirely. **Decide the final chapter structure here, once, before any drafting begins.** Changing chapter boundaries after Phase 2 is expensive (re-splits prose, re-runs reviews, re-writes demos).

Dispatch **1 subagent** (type: `general-purpose`) to read every Phase 1 artifact — the architecture survey (1.1), initial dependency graph (1.2), and all three layers of deep-dive traces (1.3) — and produce a binding `/tmp/tutorial-<timestamp>/chapter_plan.md`:

```markdown
# Chapter Plan

## Final Chapter List (N chapters, target 8-16)

### Ch 1: <Title>
- **Subtitle**: <one-sentence hook>
- **Layer**: foundations | orchestration | execution
- **Covers**: <list of concepts from Step 1.3 deep-dive traces>
- **Scope (in)**: <what this chapter teaches>
- **Scope (out)**: <what's deliberately deferred to a later chapter>
- **Prerequisite chapters**: none | [2, 3]
- **Consumed by**: [5, 7]
- **Depth**: shallow (overview) | medium (mechanism) | deep (implementation-level)
- **Target length**: <word count from the length targets in Step 1.6>
- **Primary source files**: <list>
- **Key snippets**: <references to verified citations>
- **Interactive demo anchor**: <one-line concrete idea>
- **Figure/diagram anchor**: <architecture / sequence / state / timeline>
- **Risk notes**: <e.g. "overlaps with Ch 3 — make the split crisp">

### Ch 2: ...
(one entry per chapter)

## Decisions Log
Explicitly record the non-obvious choices so the plan is auditable.

### Merges
- "Chunked prefill" + "Prefill scheduling": researched separately in 1.3 but one chapter reads better because they share the same decision point.

### Splits
- "Attention" was one unit in 1.2; becomes Ch 7 (math intuition) + Ch 8 (attention backends / FlashAttention / FlashInfer) because the second half is backend-specific and interrupts the math story.

### Drops
- "Logging infrastructure" researched but no chapter — out of scope for a how-it-works tutorial.

### Added (gap-filling)
- "What is paged memory?" — new Ch 4 as a prerequisite for KV cache, since neither 1.2 nor 1.3 flagged that readers won't have this concept.

## Final Dependency Graph
<revised from 1.2, reflecting merge/split/add decisions>
Ch1 → Ch2 → Ch3 → Ch4 → {Ch5, Ch6} → Ch7 → Ch8 → ...

## Chapter Ordering Rationale
- Opens with <Ch 1> because <reason>
- Closes with <Ch N> because <reason>
- Crossover point (advanced material begins): Ch K
```

The agent applies these rules while planning:
- **Chapter 1 is always an architecture overview — non-negotiable.** Readers
  need a map before a tour; every later chapter zooms into one region of
  this map. Ch 1's plan entry must specify:
  - **Depth**: `shallow (overview)` — no exceptions.
  - **Figure/diagram anchor**: `architecture (full-system)` — a single
    `svg.diagram` with every major component as a named node, connected
    by the data-flow paths the system actually uses.
  - **Scope (in)**: the full component list (one line per component
    stating its role), one worked end-to-end request/input example traced
    through the diagram with numbered arrows, and a pointer table
    mapping each diagram component to the chapter that covers it.
  - **Scope (out)**: every mechanism. Ch 1 never deep-dives — if a
    paragraph would belong in Ch 3 or Ch 7, it goes there.
  - **Target length**: ≤ 1,200 words (overrides the regular-chapter range).
  - **Invariant**: every named node in the Ch 1 diagram must either (a)
    be the subject of a later chapter, or (b) appear in Ch 1's explicit
    "Not covered in this tutorial" list with a one-line reason. No orphan
    boxes.
- **No chapter lives below 1,000 words of material** — if the deep-dive trace can't support that, merge into a sibling. (Ch 1 is exempt from the floor but still held to the ≤ 1,200 ceiling above.)
- **No chapter carries more than three independent concepts** — if it does, split.
- **Every chapter's prerequisites must appear earlier in the list** — the final graph is a topological sort.
- **Foundations come first, execution comes last** — respect the layer split even if a specific concept's researcher gave it a different priority.
- **Audience-gated depth** — beginner audiences get shallower chapters with fewer concepts each; expert audiences tolerate denser merges.

The agent reports back the chapter count and the path to the plan — nothing more. The plan becomes **the binding contract** for every downstream step:
- Step 1.5 Citation Gate audits snippets against the final chapter list (not the 1.2 concept list).
- Step 1.6 Consistency Bible is built per the final chapter titles, cross-reference map, and length targets.
- Step 2.2 Raw Draft, Step 2.4 Prose Pass, and Step 4.2 Demos all work from the plan.

If any downstream step discovers the plan is wrong (e.g., Ch 5 can't actually stand on its own), the orchestrator re-dispatches Step 1.4 with the failure note — **never** edits the plan ad-hoc mid-draft.

### Step 1.5: Cheap Citation Gate (run BEFORE drafting)

Every source snippet from Step 1.3 carries a `file.py:L42-L78` tag. **Verify every tag against the actual codebase now**, before any prose is written and long before HTML assembly — fixing a broken reference in research notes costs one sentence; fixing it after it's embedded in the HTML costs a re-render.

Dispatch **3 parallel subagents** (type: `general-purpose`), one per layer from Step 1.3 (Foundations / Orchestration / Execution). Each agent audits the snippets for its own layer — same split, so each agent already has the relevant mental model.

For each snippet, the agent runs:
- Read the cited file. Does the range exist?
- Does the first line of the cited range match the first line of the snippet? (Line numbers drift between releases; use a distinctive identifier from the snippet — class name, function signature, comment — to relocate if the range has shifted.)
- If the snippet contains a class/function name, does `grep -n "<name>" <file>` actually find it at the expected location?

Each agent produces a per-layer `citation_audit_<layer>.md`:
```
OK      sglang/srt/managers/scheduler.py:L142-L178   (event_loop_normal)
SHIFT   sglang/srt/layers/attention.py:L55-L88       → now at L61-L94
MISSING sglang/srt/legacy/old_router.py:L12-L30      (file deleted; use managers/router.py:L40-L58)
```

Each agent also **applies fixes in place** to the research notes for its layer: update line ranges, relocate snippets that moved, drop or replace references to deleted code. The orchestrator reads only the summary counts (OK / SHIFT / MISSING) from each agent's final report — not the individual snippet contents. Only after all three reports return clean does Step 1.6 begin.

### Step 1.6: Build the Consistency Bible (run BEFORE drafting)

Three parallel writers will produce chapters that drift without a shared reference. One will call it a "request", another a "query", another an "input". One will draw the KV cache as a grid; another as a list; another as a tree. Post-hoc consistency review is expensive — every fix touches multiple files. **Fix consistency upfront with a shared artifact**.

Dispatch **1 subagent** (type: `general-purpose`) to synthesize all Phase 1 outputs into a single `/tmp/tutorial-<timestamp>/consistency.md`. The agent reads: the architecture survey (Step 1.1), the dependency graph (Step 1.2), the deep-dive traces per layer (Step 1.3), the gap list (Step 1.4), the audited citations (Step 1.5). It produces:

```markdown
# Consistency Bible

## Canonical Terminology
For every important concept, the ONE noun/verb to use. Writers and reviewers
must enforce these. List the aliases that are forbidden.

| Canonical | Aliases to avoid | Defined in |
|---|---|---|
| request | query, input, prompt (when referring to the flow unit) | Ch 1 |
| KV cache | key-value cache, kv pool, attention cache | Ch 3 |
| prefill | prompt-phase, input-phase, context-phase | Ch 5 |
| decode | generation-phase, output-phase, token-phase | Ch 5 |
| scheduler | dispatcher, orchestrator, manager (ambiguous) | Ch 2 |
| ... | ... | ... |

## Abbreviations (expand on first use in every chapter)
| Abbrev | Full form |
|---|---|
| LLM | large language model |
| KV  | key-value (attention) |
| FSM | finite state machine |
| ... | ... |

## Cross-Reference Map
Which chapter owns each concept's introduction. When another chapter mentions
the concept, it must either define it briefly in place OR link to this chapter.

| Concept | Defined in |
|---|---|
| Radix tree | Ch 4 |
| Continuous batching | Ch 6 |
| Paged memory | Ch 3 |
| ... | ... |

## Pseudocode Conventions
- Variable names: use the project's actual names where possible (req, batch, seq_len).
  Never invent cute variables (foo, my_thing).
- Function signatures: match the real codebase's naming (snake_case for Python,
  camelCase for TS, etc.).
- Comment marker for annotations in snippets: ALWAYS `# ←` (never `//`, `→`, or plain `#`).

## Notation Conventions
Consistent visual metaphors across diagrams.
- KV cache: rendered as a grid of cells (rows = requests, cols = token positions).
- Request lifecycle: rendered as a horizontal flow (arrows left→right).
- Scheduling decisions: rendered as a timeline with bars per request.
- Tree structures: rendered as nested SVG nodes, roots at the top.

## Reusable Analogies
If an analogy appears in one chapter, no other chapter introduces a competing
one for the same concept.

| Concept | Analogy (pick ONE) |
|---|---|
| Scheduler | restaurant host seating parties at tables (Ch 2) |
| KV cache | sticky notes with partial work preserved (Ch 3) |
| Radix tree | filing cabinet with shared folder prefixes (Ch 4) |
| ... | ... |

## Glossary (reader-facing)
Plain-English, one-line definitions for every technical term the tutorial uses
without first defining it in prose. These power hover-to-define tooltips in
the rendered HTML (Step 4.2 reader components). One entry per canonical term —
aliases are handled by the Canonical Terminology table, not duplicated here.

Definitions must be:
- **≤ 140 characters**, so the tooltip fits in one line on mobile.
- **self-contained** — no terms that would themselves need a tooltip.
- **concrete over formal** — "a table the model looks up previously computed
  attention values in" beats "cache of intermediate activations".

| Term | Plain-English definition |
|---|---|
| KV cache | Table of previously computed attention values the model reuses instead of recomputing each token. |
| Radix tree | A tree that shares common prefixes between keys, so matching two strings that start the same way costs only the differing tail. |
| Prefill | The phase where the model reads the user's prompt in one big batch before generating anything. |
| Decode | The phase where the model generates one token at a time, feeding each one back in. |
| ... | ... |

Every term that appears in a chapter's prose and is not introduced in-chapter
must exist in this table. The Step 4.3 gate greps `class="glossary"` spans
against this list.

## Chapter Length Targets
Approximate word counts so no chapter balloons or starves. Deviate only with justification.

| Chapter | Target words |
|---|---|
| Ch 1 (intro) | 800-1200 |
| Regular chapter | 1500-2500 |
| Complex chapter (e.g. attention) | 2500-3500 |

## Style Rules
- Headings: sentence case, not Title Case.
- Numerals: "two" for <10 in prose, digits for technical/numeric contexts.
- Code voice: present tense ("the scheduler picks the next batch"), not past or future.
- Callouts: info = neutral explainer, success = key takeaway, warning = common pitfall.
  Never use info for warnings or vice versa.
- **No wall-of-text sections**: within a single `###` subsection, no more
  than **~8 consecutive paragraphs** without *any* visual interruption — a
  code snippet, SVG diagram, demo, callout, table, or list. This is a
  page-level floor, not per-paragraph pacing — engineering prose explaining
  *why* an algorithm works is welcome; an unbroken essay is not.
  Enforced by Step 2.5 polish and Step 4.3 gate.
- **Metaphor fit-for-purpose**: each analogy in the Reusable Analogies table
  must map tightly to the specific concept it pairs with. Generic metaphors
  (a recipe, a factory, a conveyor belt) applied to multiple unrelated
  concepts are forbidden — once "factory" is used, it's used for one concept
  only; find a different metaphor for the next one.
```

The agent produces this file and reports only its path back. The orchestrator does not read the contents — every downstream writer/reviewer does.

**Enforcement**:
- Step 2.2 writers receive `consistency.md` and obey it.
- Step 2.3 per-chapter reviewers add a `CONSISTENCY` line to their report — did the chapter violate any bible rule? Reviewers fix violations in place.
- Phase 3 Round 1 cross-chapter reviewers use the bible as the source of truth (the bible is the target; drift is a violation of the bible, not a debate between chapters).

---

## Phase 2: Draft Writing

### Step 2.1: Chapter Template

Every chapter follows this structure:

```markdown
## N. Chapter Title
Subtitle: one-sentence hook.

### The Problem
What real-world problem does this concept solve?
Start with the pain — what breaks, what's slow, what's wasteful without this?

### The Idea (Simplified)
Core idea in 2-3 paragraphs.
Use analogies if AUDIENCE=beginner.

### [If applicable] What Is <Technique>?
When a chapter introduces a non-obvious data structure, algorithm, or technique
(radix tree, ring buffer, FSM, speculative execution, etc.), explain IT before
explaining how the codebase uses it. The reader needs to understand the tool
before seeing it applied.

Structure as:
1. What is it in general? (1-2 paragraphs, CS-agnostic explanation)
2. Why is it the right fit here? (compare to alternatives the reader might expect)
3. Then transition into how the codebase implements it.

Example for RadixAttention chapter:
  - First explain: "A radix tree (trie) is a compressed prefix tree where..."
  - Then explain: "Why not a hash table? Because hash lookup is O(N) per prefix
    length — you must hash the entire sequence to check if it's cached. A radix
    tree matches incrementally: once you've matched N tokens, checking token N+1
    is O(1)."
  - Then: "Here's how SGLang's RadixCache implements this..."

### Why This, Not That?
Every chapter should answer: "Why did the authors choose THIS approach over
the obvious alternative?" This is often the most valuable part of a tutorial
because it builds engineering intuition, not just knowledge.

Structure as a comparison:
  | Approach | Pros | Cons | When to use |
  | The chosen approach | ... | ... | (this codebase) |
  | The obvious alternative | ... | ... | (when X is different) |

Examples of "Why this, not that?" questions worth answering:
  - Why radix tree, not hash table for prefix caching?
  - Why separate processes via ZMQ, not async tasks in one process?
  - Why ring shadows instead of drop shadows? (design)
  - Why chunked prefill instead of just priority queuing?
  - Why CUDA graph replay instead of torch.compile?
  - Why FSM-based constrained decoding instead of retry-on-invalid?

### How It Works (Detailed)
Step-by-step technical walkthrough.
Include pseudocode showing key algorithms.
Reference actual data structures from the codebase.

### Source Code (from the real codebase)
Embed actual source code snippets from Phase 1.
- Present AFTER prose for beginners (context first)
- Lead WITH code for experts (annotate interesting parts)
- Trim to 10-40 lines per snippet
- Always include `# source:` tag with file path

### Quick Check
One optional scenario-based quiz per chapter, placed after *Key Takeaway*.
Tests whether the reader can **apply** the chapter, not whether they memorized
a definition. The question describes a realistic situation; the answer
reveals which part of the system they'd touch, or why one alternative wins.

Good:
> A new request arrives while the batch is mid-decode. Which file decides
> whether it waits for the next step or gets chunked in immediately?
> <br>**Answer**: `schedule_batch.py` — `add_new_request()` at L88. Chunked
> prefill (Ch 6) lets this request ride along instead of waiting.

Bad (tests recall, not application):
> What does KV cache stand for?

Rules:
- Offer **2–4 answer choices**, not free text — the reader clicks, sees the
  result, learns why.
- Every answer (right or wrong) gets a one-line explanation. Wrong answers
  should be plausible — they teach what *not* to think.
- Cite a real `file:line` in the correct answer's explanation.
- Omit the block if no good scenario exists; never pad.

Rendered as the `quiz` reader component (Step 4.2).

### Further Reading
Link to the original papers and references that introduced the techniques
used in this chapter. Not every chapter needs this — only chapters where the
technique has a clear research origin (most do in systems/ML codebases).

Every entry **must** include a real, resolvable URL — arXiv, DOI, project
homepage, or canonical blog post. If you can't find a URL, don't cite the
paper; a reference the reader can't click is dead weight. No bare text
citations.

Format each entry as a rendered markdown link (not a placeholder):

```markdown
- [Fast Inference from Transformers via Speculative Decoding](https://arxiv.org/abs/2211.17192) — Leviathan et al., 2023. Introduced the draft-then-verify paradigm.
```

The literal string `[arXiv link]` or `<arxiv>` must never appear in the
output — these are placeholders from the template, not content.

### Interactive Demo
Description of the interactive visualization.

### Key Takeaway
One callout box summarizing the essential insight.
```

### Two-Pass Chapter Pipeline (overview)

Drafting happens in **two passes** with a review between each:

```
Pass 1: RAW   → 2.2 Raw Draft (bullets/facts/snippets)     →  2.3 Raw Review (facts, citations, bible)
Pass 2: PROSE → 2.4 Prose Pass (approved raw → paragraphs) →  2.5 Prose Polish (flow, tone)
```

Why split it: fact-checking prose is expensive — reviewers must distinguish "wrong fact" from "awkward sentence." Fact-checking bullets is trivial. By the time prose exists, every claim, citation, and consistency rule has already been vetted; the prose pass is pure writing, and its review is cosmetic.

All four steps reuse the Foundations / Orchestration / Execution layer split for writers and the one-reviewer-per-chapter pattern for reviews. Writers always write to disk; reviewers always edit in place.

### Step 2.2: Raw Draft (parallel by layer)

Dispatch **3 parallel subagents** (same layer split as Step 1.3). Each writes **raw content — not prose** for its chapters, one file per chapter:

```
/tmp/tutorial-<timestamp>/raw/ch01.raw.md
/tmp/tutorial-<timestamp>/raw/ch02.raw.md
...
/tmp/tutorial-<timestamp>/raw/chNN.raw.md
```

Raw content is **structured bullets, facts, and fragments** — no paragraphs, no flow. Every required template section appears as a heading with bullets underneath:

```markdown
## N. <Chapter Title>
Subtitle: <one-line hook>

### The Problem
- <pain point 1, one bullet>
- <pain point 2>
- Without this: <concrete consequence>

### The Idea
- Core mechanism: <one sentence>
- Key insight: <one sentence>
- Analogy (from consistency.md): <reference>

### How It Works
- Step 1: <fact> (source: scheduler.py:L42)
- Step 2: <fact> (source: scheduler.py:L58)
- Key data structure: <Class name> — fields: {a, b, c}
- Complexity: O(...)

### Source Code Snippets
- SNIPPET: scheduler.py:L42-L78  (primary loop, verified in Step 1.5)
- SNIPPET: schedule_batch.py:L12-L40  (data structure)

### Why This, Not That?
- Chosen: <approach> because <reason>
- Alternative: <approach> — pros/cons
- Verdict: <one line>

### Interactive Demo
- Type: <from common types table>
- User input: <what triggers it>
- Animation: <what changes over time>
- Stats shown: <counter 1, counter 2>
- Insight delivered: <one line>

### Papers & References
- TITLE: "<Paper title>"
  AUTHORS_YEAR: <Leviathan et al., 2023>
  URL: <https://arxiv.org/abs/2211.17192>   # REQUIRED — arXiv, DOI, or canonical page. Omit entry if no URL exists.
  SUMMARY: <one-line contribution>
  IMPLEMENTED_IN: <files>

### Key Takeaway
- <one sentence>

### Quick Check (optional — omit if no good scenario)
- SCENARIO: <realistic one-sentence situation>
- CHOICES:
  - A) <plausible-wrong>
  - B) <correct>
  - C) <plausible-wrong>
- CORRECT: B
- EXPLANATIONS:
  - A: <one line on why it's tempting but wrong>
  - B: <one line; must cite file:line>
  - C: <one line on why it's tempting but wrong>
```

Each writer receives: **the binding chapter plan from Step 1.4** (title, subtitle, scope in/out, prerequisites, depth, target length for each assigned chapter), the verified citation list (Step 1.5), the Consistency Bible (Step 1.6), the deep-dive traces for their concepts, the target AUDIENCE level. Writers do not invent their own chapter titles, subtitles, or scope boundaries — those come from the plan. Writers **do not write prose** and must not — a writer that drifts into paragraphs fails the review in Step 2.3.

Each writer reports back the list of files written and any citations that turned out to be unusable.

### Step 2.3: Raw Review (parallel, one per chapter — fires as raw files land)

The moment a Step 2.2 writer reports a file complete, dispatch **1 subagent per raw chapter file**. Don't wait for all three writers to finish. Reviewing bullets is fast and cheap; do it in a tight feedback loop.

Each reviewer receives:
- Path to its single `.raw.md` file
- The Consistency Bible (Step 1.6)
- The verified citation list (Step 1.5)
- The deep-dive trace for this chapter's concept
- The concept dependency graph (for forward-ref checks)

Each reviewer checks:

1. **Structure**: every required template heading is present, each with at least one bullet.
2. **Factual accuracy**: every technical claim matches the deep-dive trace or the actual code. Class names, function names, algorithms, complexities all correct.
3. **Citation fidelity**: every `file.py:L42-L78` tag matches the verified list. Every SNIPPET's line range still exists. No snippets invented.
4. **Consistency bible**: every canonical term is used; no forbidden aliases. Abbreviations expanded on first mention. Forward-refs obey the cross-reference map. Pseudocode conventions followed.
5. **Demo concreteness**: the demo spec names a type, an input, an animation, and specific stats — not "a visualization."
6. **No prose**: raw files are bullets only. If a writer started drafting paragraphs, kick the content back to bullet form.

Reviewer **applies fixes in place** and returns a ≤120-word structured report:
```
CHAPTER: ch05.raw.md
STRUCTURE: ok
FACTS: fixed 1 (said O(n), actual O(n log n))
CITATIONS: fixed 2 (L142→L148 drift, L201→L208 drift)
BIBLE: fixed 3 ("query"→"request" ×2, expanded KV on first use)
DEMO: ok
PROSE-LEAK: fixed (converted 2 paragraphs back to bullets)
UNFIXABLE: 0
```

If a reviewer hits `UNFIXABLE`, the orchestrator re-dispatches that chapter's original writer with the failure report. Only when every chapter returns a clean raw review does Step 2.4 begin.

### Step 2.4: Prose Pass (parallel by layer)

Dispatch **3 parallel subagents** (same layer split) to turn approved raw content into polished chapter prose, one file per chapter:

```
/tmp/tutorial-<timestamp>/ch01.md
/tmp/tutorial-<timestamp>/ch02.md
...
/tmp/tutorial-<timestamp>/chNN.md
```

Each agent reads its layer's reviewed `.raw.md` files and produces finished prose. The job is **writing, not research**: every fact, citation, and term is already locked. The agent must not introduce new claims, new source snippets, or new terminology — doing so means a factual re-review is required and that's expensive. If a prose agent discovers a gap, it logs it (`GAP: ch07 — need one more example of X`) and keeps writing; the orchestrator decides whether to re-run the raw pass for that chapter or proceed.

Each agent receives: its reviewed raw files, the Consistency Bible, the chapter template, the target AUDIENCE level, the chapter length targets from the bible. Agents report back the list of files written and any gap logs.

### Step 2.5: Prose Polish (parallel, one per chapter — fires as prose files land)

Dispatch **1 subagent per prose chapter file** as each is written. This review is intentionally **lightweight** — facts were already vetted in Step 2.3.

Each reviewer checks only:
- **Flow and tone**: does it read well at the AUDIENCE level? Awkward sentences, dense paragraphs that should be split, abrupt transitions.
- **Length**: within the bible's chapter target, ±20%.
- **Prose faithfulness**: did the prose accidentally contradict the approved raw content? (If yes, prose wins ONLY for wording; any factual contradiction must be fixed to match the raw.)
- **Callout placement**: info vs success vs warning used per bible rules.

Reviewer **applies fixes in place** and returns a ≤80-word report:
```
CHAPTER: ch05.md
FLOW: smoothed 3 transitions
LENGTH: 2100 words (target 1500-2500) ok
FAITHFULNESS: ok
CALLOUTS: ok
```

Only after every chapter has a clean prose-polish report does Phase 3 begin.

---

## Phase 3: Review

### Round 1: Cross-Chapter Consistency

Per-chapter reviews (Step 2.3) already caught every per-chapter accuracy issue. Round 1 now focuses only on problems no single-chapter reviewer can see:

Dispatch **2 parallel subagents**, each reading half the chapters in order:
- Terminology drift — the same concept called three different names across chapters
- Redundant explanations — the same idea explained from scratch in two chapters
- Broken cross-references — "as covered in Ch 3" where Ch 3 doesn't cover it
- Inconsistent notation or code style between chapters
- Chapter-to-chapter transitions that don't follow the dependency graph

### Round 2: Pedagogy

Dispatch **1 subagent** (type: `general-purpose`) role-playing the target AUDIENCE level. The agent reads every chapter file from `/tmp/tutorial-*/ch*.md` in order, simulating a first-time reader, and produces a `pedagogy_review.md` listing:
- Undefined terms used before introduction (with chapter:paragraph refs)
- Leaps in complexity where scaffolding is missing
- Chapters lacking "why should I care" motivation
- Analogies that don't land for the target audience
- Suggested diagram insertions where prose is carrying too much load

The agent then **applies fixes directly** to the chapter files — adding forward-reference definitions, extra paragraphs, or `What Is X?` sections where flagged. The orchestrator reads only the summary (issue count and diff count) from the agent's report. Running this as a dedicated agent matters because the orchestrator's context is saturated with code, citations, and HTML; a fresh agent reads the draft the way a real reader would.

---

## Phase 4: Interactive HTML Generation

### Step 4.0: Two-Stage Assembly (Shell → Chapters)

Never have a single agent write the entire HTML in one pass. Split the work:

**Stage 1 — Shell agent**. Dispatch one subagent to write ONLY the HTML shell to `$OUTPUT`:
- `<head>` with meta tags
- Full `<style>` block (CSS custom properties from the design system, all layout/component styles, demo primitives)
- `<body>` layout: sidebar, main content container, hamburger, overlay
- An empty `<main id="chapters"></main>` placeholder — chapters will be injected here
- `<script>` block with: navigation JS, `<!-- CHAPTER_SCRIPTS -->` marker for demo JS, RNG seeding helper, init code
- No chapter prose. The shell is ~400-800 lines.

**Stage 2 — Assembler (main loop, not a subagent)**. The orchestrator reads `/tmp/tutorial-*/chNN.md` one file at a time, converts each to the chapter HTML fragment (see Step 4.2), and appends it into `<main id="chapters">` using Edit tool inserts. After each insert, also append the chapter's demo JS block before the `<!-- CHAPTER_SCRIPTS -->` marker.

Why the orchestrator, not a subagent: chapters accumulate, context doesn't. The orchestrator processes one chapter per tool call; the full assembled HTML never needs to be held in any single context. Subagents are appropriate for writing chapter prose (parallel, independent) but assembly is inherently sequential and stateful — use direct tool calls.

Checkpoint after every 3 chapters: verify the file still parses (grep for balanced tags, no stray `<!-- CHAPTER_SCRIPTS -->` duplication).

### Step 4.1: HTML Shell

Write a **single self-contained HTML file** with zero external dependencies. Must work by double-clicking in a browser.

```
┌─────────────────────────────────────────────┐
│  Sidebar (chapter nav, progress tracking)   │
├─────────────────────────────────────────────┤
│  Main content area                          │
│  ┌─────────────────────────────────────┐    │
│  │  Chapter title + subtitle           │    │
│  │  Prose sections                     │    │
│  │  ┌──────────────────────────────┐   │    │
│  │  │  INTERACTIVE demo box        │   │    │
│  │  │  (animations, controls,      │   │    │
│  │  │   stats counters)            │   │    │
│  │  └──────────────────────────────┘   │    │
│  │  Source code blocks (highlighted)   │    │
│  │  Callout boxes (info/warn/success)  │    │
│  │  Chapter navigation (prev/next)     │    │
│  └─────────────────────────────────────┘    │
└─────────────────────────────────────────────┘
```

**Required UI features**:
- Sidebar with numbered chapter list, active/completed state, progress bar
- Mobile responsive (sidebar collapses to hamburger menu)
- Smooth chapter transitions (fade + slide animation)
- Code blocks with keyword highlighting (inline, no external deps)
- Chapter-to-chapter navigation (prev/next buttons)
- **Bounded scroll inside the content column — NOT at page level.** The browser window itself must never scroll. Layout:
  ```css
  html, body { height: 100vh; overflow: hidden; margin: 0; }
  .layout { display: flex; height: 100vh; }
  .sidebar { width: var(--sidebar-width); height: 100vh; overflow-y: auto; position: fixed; }
  .content { margin-left: var(--sidebar-width); flex: 1; height: 100vh; overflow-y: auto; }
  ```
  On chapter switch, the content column resets to top: `document.querySelector('.content').scrollTop = 0`. Never `window.scrollTo(0, 0)` — that's a page-level scroll that shouldn't exist here. Mobile: sidebar becomes an overlay, `.content` still scrolls internally.
- **Palette closure**: no color outside the active DESIGN.md's palette may appear in any emitted CSS rule, inline `style=`, or SVG attribute. Grep-checkable: `fill="#`, `stroke="#`, `color:#`, `background:#` with a hex value must match a DESIGN.md token.
- **Dark-mode gating**: emit a `@media (prefers-color-scheme: dark)` block ONLY if DESIGN.md defines a dark palette. If it doesn't, lock the page with `color-scheme: light` (or `dark`, whichever the brand is) in `:root`. Never invent dark tokens to fill a missing palette.
- All styling via CSS custom properties so the design system is a single block of overrides.
- **Include the `svg.diagram` CSS scaffold** from Step 4.2's "Diagram Theme Consistency" section verbatim — every inline SVG diagram across all chapters inherits from it, so the shell MUST ship it once. Without this block, per-chapter diagrams will hard-code colors and drift from the theme.

**Apply the resolved design system** from the Parse Arguments step. Use the design tokens for:
- CSS custom properties (colors, surfaces, text, borders)
- Typography (headline font family, body font family, weights, line-heights)
- Border radius scale (from the design system's component stylings)
- Shadow/elevation system (ring shadows vs drop shadows)
- Button styles, callout colors, interactive element styling
- Do's and Don'ts as hard rules

**Source code block styling** — must be visually distinct from pseudocode:

```html
<div class="source-code">
  <div class="source-header">
    <span class="source-tag">SOURCE CODE</span>
    <span class="source-path">path/to/file.py:L42-L78</span>
  </div>
  <pre><code><!-- syntax-highlighted real code --></code></pre>
</div>
```

- Left border accent (3px solid, using the design system's brand/accent color)
- Header bar with `SOURCE CODE` tag + file path, sitting on the same code surface — inherits the `--code-bg` token, not a hardcoded dark fill
- Inline annotations (`# ←`) in dimmer italic font, using the accent color
- **Code block surface follows the design system.** Resolve `--code-bg` from the active DESIGN.md. On light/parchment brands (Claude, editorial) `--code-bg` is a warm cream surface, not dark. On dark brands it's a deeper dark. Never hardcode `#141413` or any other literal — always go through the token. The same applies to `--code-header-bg`, `--code-text`, and every syntax-highlight class color.

**Claude built-in code-block palette (light)** — use these exact values when `--design claude` or no flag is specified. Designed for the warm parchment canvas; every color is contrast-tested against the cream code surface.

```css
/* Surfaces */
--code-bg: #f0eee6;          /* Border Cream — warm cream code surface, distinct from chapter bg */
--code-header-bg: #e8e6dc;   /* Warm Sand — slightly darker for the source-header bar */
--code-text: #3d3d3a;        /* Dark Warm — primary code text */
--code-border: #d1cfc5;      /* Ring Warm — soft border for code blocks */

/* Inline code (`foo` inside paragraphs) */
--code-inline-bg: #faf9f5;   /* Ivory — lightest surface for inline pill */
--code-inline-text: #c96442; /* Terracotta — accented for quick identification */

/* Syntax highlighting — each tuned for contrast against #f0eee6 */
.kw  { color: #a84c2d; font-weight: 500; }  /* keywords: deeper terracotta */
.fn  { color: #6b4a80; }                    /* functions: muted purple */
.cls { color: #6b4a80; font-weight: 500; }  /* class names: muted purple, heavier */
.st  { color: #4a6b3e; }                    /* strings: darker sage green */
.num { color: #8b5e2f; }                    /* numbers: warm tan */
.cm  { color: #87867f; font-style: italic; }/* comments: stone gray */
.an  { color: #b5562f; font-style: italic; }/* annotations (# ←): deep coral italic */
.op  { color: #5e5d59; }                    /* operators: olive gray */
.dec { color: #a84c2d; }                    /* decorators: deeper terracotta */
.bool,.null,.self { color: #6b4a80; font-style: italic; }
```

Do not swap in the dark-background syntax colors (`#d97757` coral keywords, `#c9a0dc` lavender functions, `#a8c4a0` sage strings) — those were tuned for a black code surface and wash out against parchment. The rule of thumb: on light `--code-bg`, every syntax color needs ≥4.5:1 contrast against the cream; the tokens above already pass.

**Callout boxes**: use the design system's accent for info, success color for success, accent for warning

**"What Is X?" explanation boxes** — when a chapter introduces a non-obvious technique:

```html
<div class="callout callout-info">
  <strong>What is a radix tree?</strong> A radix tree (also called a compressed
  trie) stores sequences by sharing common prefixes. Each node holds a segment
  of the sequence, not a single character...
</div>
```

Style these with the info callout color and a bold lead question. Place them BEFORE the "How It Works" section so the reader has the prerequisite knowledge.

**"Why This, Not That?" comparison tables** — for design decision explanations:

```html
<table class="compare-table">
  <thead><tr><th>Approach</th><th>Lookup Cost</th><th>Prefix Sharing</th><th>Verdict</th></tr></thead>
  <tbody>
    <tr class="chosen"><td>Radix tree</td><td>O(1) incremental</td><td>Natural</td><td>Chosen</td></tr>
    <tr><td>Hash table</td><td>O(N) per lookup</td><td>None</td><td>Too slow for long prefixes</td></tr>
  </tbody>
</table>
```

Style `.compare-table` with the design system's card surface, border, and radius. Highlight the `.chosen` row with a subtle accent background. These tables are high-value — they build engineering intuition, not just knowledge.

**"Further Reading" paper references** — link to the original research for each technique:

```html
<div class="further-reading">
  <h4>Further Reading</h4>
  <ul>
    <li>
      <a href="https://arxiv.org/abs/2211.17192" target="_blank" rel="noopener">
        Fast Inference from Transformers via Speculative Decoding
      </a>
      <span class="paper-meta">Leviathan et al., 2023</span>
      — Introduced the draft-then-verify paradigm that SGLang implements in its speculative module.
    </li>
  </ul>
</div>
```

Style `.further-reading` as a subtle card at the bottom of each chapter (before the key takeaway callout). Use the design system's secondary surface with a border. Paper links use the accent color. `.paper-meta` is dimmer/smaller text for authors and year. This section is the bridge between "how the code works" and "why the field works this way."

### Step 4.2: Build Interactive Demos (parallel by layer)

Dispatch **3 parallel subagents** (type: `general-purpose`), split by the same Foundations / Orchestration / Execution layers used in Step 1.3 and Step 2.2. Each agent writes the demo HTML fragment + JS for its chapters directly to disk, one file per demo:

```
/tmp/tutorial-<timestamp>/demos/ch01.demo.html   (the <div class="interactive">...</div> block)
/tmp/tutorial-<timestamp>/demos/ch01.demo.js     (the JS that drives it — uses rand(), not Math.random())
...
```

Each agent receives: the chapter prose for its layer, the design system tokens, the seeded-RNG helper signature, and the demo pattern below. Agents report back only the list of files written. The orchestrator injects them during Stage 2 assembly (Step 4.0) — the HTML block goes inside the chapter's section, the JS block goes before the `<!-- CHAPTER_SCRIPTS -->` marker.

Every demo follows this pattern:

```html
<div class="interactive">
  <span class="label">TRY IT</span>
  <h4>Demo Title</h4>
  <p class="demo-desc">What to do and what to watch for.</p>
  <div class="controls"><!-- buttons, sliders --></div>
  <div class="viz"><!-- visualization area --></div>
  <div class="stat-row">
    <div class="stat-box"><div class="stat-value">0</div><div class="stat-label">Label</div></div>
  </div>
</div>
```

**Demo design principles**:
1. **Input → Visual Feedback → Insight**. Every demo must have a clear "aha moment."
2. **Immediate response**. Animations start within 100ms of user action.
3. **Stat counters** make abstract concepts concrete (e.g., "Cache hit rate: 73%").
4. **Reset buttons** let users replay. No dead-end states.
5. **Progressive complexity**: first interaction is a single button click.
6. **Determinism: seed every RNG**. Demos that use randomness (arrival times, acceptance rates, token samples, cache evictions) must be reproducible across reloads — two users comparing notes should see the same numbers. Use a tiny seeded PRNG (mulberry32 is ~4 lines); the shell must include it once in the global `<script>` block and every demo must draw from `rand()` (seeded) instead of `Math.random()`. Reset buttons re-seed from a fixed value so the "same" run really is the same run.
7. **Prefer inline SVG diagrams for structural/spatial concepts.** For anything with geometry — architecture, memory layout, dataflow, tree structure, attention patterns, pipeline stages — a labeled SVG diagram is almost always clearer than prose or a stack of `<div>`s. Author SVG directly inline (no external libs, no Mermaid): `<svg viewBox="0 0 800 400">` with `<rect>`, `<circle>`, `<path>`, `<text>`, and `<line>`. Use CSS custom properties (`fill="var(--accent)"`) so the diagram inherits the design system. Animate via `<animate>` or by toggling classes from JS on interaction. Every chapter that introduces a data structure, a pipeline, or a cross-component relationship should include at least one SVG diagram — either as the primary demo or as a standalone figure alongside the demo.

```js
// Include once in the shell's <script> block — every demo uses rand() not Math.random()
let __seed = 0x9e3779b9;
function reseed(s) { __seed = (s >>> 0) || 0x9e3779b9; }
function rand() {
  let t = __seed = (__seed + 0x6d2b79f5) >>> 0;
  t = Math.imul(t ^ (t >>> 15), t | 1);
  t ^= t + Math.imul(t ^ (t >>> 7), t | 61);
  return ((t ^ (t >>> 14)) >>> 0) / 4294967296;
}
```

**Diagram Theme Consistency — keep SVG native to the tutorial**:

Diagrams must feel like part of the page, not screenshots pasted in. Use the same tokens the rest of the UI uses. The shell's `<style>` block must include this reusable scaffold once; every diagram then inherits it by putting its SVG inside `<svg class="diagram">`.

```css
/* Add to the shell's <style> block — every diagram inherits these.
   :where() selectors have zero specificity, so inline fill/stroke attributes
   on any element still override these defaults (critical — see Known Traps). */
svg.diagram { font-family: inherit; display: block; max-width: 100%; height: auto; overflow: visible; }
svg.diagram :where(text)        { fill: var(--text-bright); font-size: 13px; font-weight: 500; }
svg.diagram :where(.label-sub)  { fill: var(--text-secondary); font-size: 11px; font-weight: 400; }
svg.diagram :where(.node)       { fill: var(--bg-tertiary); stroke: var(--border-strong); stroke-width: 1.5; rx: 8; }
svg.diagram :where(.node-accent){ fill: var(--accent); stroke: var(--accent); }
svg.diagram :where(.node-accent) ~ text,
svg.diagram :where(.on-accent)  { fill: var(--bg-secondary); }   /* readable text on accent fill */
svg.diagram :where(.edge)       { stroke: var(--text-secondary); stroke-width: 1.5; fill: none; }
svg.diagram :where(.edge-active){ stroke: var(--accent); stroke-width: 2.5; }
svg.diagram :where(.arrowhead)  { fill: var(--text-secondary); }
svg.diagram :where(.arrowhead-active) { fill: var(--accent); }
svg.diagram :where(.lane)       { fill: var(--bg-secondary); stroke: var(--border); stroke-width: 1; }
svg.diagram :where(.info)    { fill: var(--blue);   stroke: var(--blue); }
svg.diagram :where(.success) { fill: var(--green);  stroke: var(--green); }
svg.diagram :where(.warning) { fill: var(--orange); stroke: var(--orange); }
svg.diagram :where(.error)   { fill: var(--red);    stroke: var(--red); }
svg.diagram :where(.muted)   { opacity: 0.35; }
svg.diagram :where(.pulse)   { animation: diagram-pulse 1.2s ease-in-out infinite; }
@keyframes diagram-pulse { 50% { opacity: 0.5; } }
/* Every diagram must ship this <defs> arrowhead marker reference once per SVG */
```

**Token mapping — non-negotiable**:

| SVG attribute | Use | Never |
|---|---|---|
| Node fill | `var(--bg-tertiary)` or `var(--accent)` for emphasis | Hard-coded `#fff` or `#000` |
| Node stroke | `var(--border-strong)` | Hard-coded gray |
| Label text fill | `var(--text-bright)` (primary) or `var(--text-secondary)` (annotations) | Hard-coded black |
| Connector stroke | `var(--text-secondary)`, promote to `var(--accent)` on active | Hard-coded gray |
| Semantic colors | `.info .success .warning .error` classes (match the callout palette) | Inventing new greens/reds |
| Font family | Inherit from `body` via `font-family: inherit` — never set explicitly on `<text>` | Declaring a different font |
| Corner radius | `rx="8"` for nodes (matches button radius from the design system) | Sharp corners unless the design system uses them |
| Stroke width | `1.5` default, `2.5` for active/emphasis | `1` (too thin on retina) or `≥3` (too heavy) |

**Diagram Layout Discipline — non-negotiable**:

LLM-authored SVG has no font-metric access. Agents guess text width from character count, and the guess is almost always too narrow. Four rules kill the overlap bugs this causes:

| Rule | How to apply it | Why |
|---|---|---|
| **Box ≥ text + padding** | `rect width ≥ (longest_label_chars × 7) + 32`, `height ≥ (line_count × 20) + 16`. Use the longest label in the box as the measuring stick — *not* the average. | Default 13px `font-size: 13px` on `<text>` renders ~6.5–7px per char in a sans-serif stack. 32px padding = 16px each side. |
| **Wrap long labels** | Any label > 14 chars breaks into `<tspan x="..." dy="..."/>` lines of ≤ 14 chars each; `dy="18"` per new line; box height grows to `(lines × 20) + 16`. | One-line text shrunk to fit clips more often than multi-line text rendered at normal size. |
| **Halo text that crosses an edge** | For any `<text>` placed on top of a `<path class="edge">` or between nodes, emit a matching `<rect class="text-halo">` *before* the `<text>` element in z-order, sized to `(chars × 7 + 10) × 18`, centered on the same coords. | SVG has no text-outline primitive that inherits theme tokens; a halo rect is the cheap, universal fix. |
| **Gutter between nodes** | ≥ **24px horizontal**, ≥ **16px vertical** between any two sibling `<rect>` nodes. Add to `viewBox` so nothing touches the edge either — pad the viewBox by **24px on all sides** after computing the content bounding box. | Two nodes that visually touch read as one merged node. Labels at the edge clip. |

Shell scaffold adds one more class (emit with the rest of `svg.diagram` CSS):
```css
svg.diagram :where(.text-halo) { fill: var(--bg); stroke: none; rx: 4; }
```

**Sizing worked example** — a node containing the label "Scheduler loop":
- longest label chars = 14
- width = `14 × 7 + 32 = 130` → round up to `140`
- height = `1 × 20 + 16 = 36` → round up to `40`
- emit `<rect class="node" x="..." y="..." width="140" height="40" rx="8"/>`, then `<text x="..." y="..." text-anchor="middle" dominant-baseline="middle">Scheduler loop</text>`.

**Wrap worked example** — a node containing "Continuous batching scheduler":
- Total 29 chars, > 14 → wrap: `"Continuous"` / `"batching scheduler"` (longest line 18 chars)
- width = `18 × 7 + 32 = 158` → round up to `160`
- height = `2 × 20 + 16 = 56` → round up to `60`
- emit
  ```html
  <rect class="node" x="..." y="..." width="160" height="60" rx="8"/>
  <text x="..." y="..." text-anchor="middle">
    <tspan x="..." dy="0">Continuous</tspan>
    <tspan x="..." dy="18">batching scheduler</tspan>
  </text>
  ```

**Halo worked example** — an edge label "backpressure" sitting on top of a `<path class="edge">`:
```html
<rect class="text-halo" x="192" y="110" width="96" height="18"/>
<text x="240" y="124" text-anchor="middle" class="label-sub">backpressure</text>
```
The halo sits *before* the text, so the text paints on top of it; the halo sits *after* the edge path, so it masks the line underneath the label. Order matters.

**Do not**:
- Resize text to fit an undersized box (`font-size: 9px` to squeeze "Continuous batching" into a 100px rect). The gate rejects `<text>` with `font-size` below 11px.
- Rotate labels to fit them. A readable diagram is worth an extra 30px of width.
- Use more than one `<tspan>` line per label without also growing the box height by the formula above.

**Interactivity** mirrors the rest of the UI: add `.edge-active` / `.node-accent` by toggling classes from JS on user interaction — never by mutating inline `fill`/`stroke` attributes. Hover states should use CSS `:hover` on the SVG group, not JS listeners, whenever the interaction is purely visual.

**Do not** use `<foreignObject>`, Mermaid, canvas rasters, or base64-embedded PNGs — all of these break theme inheritance. If a diagram requires text wrapping across multiple lines, use multiple `<tspan>` elements.

**Example — a two-node flow, 8 lines, fully themed**:
```html
<svg class="diagram" viewBox="0 0 320 80">
  <defs><marker id="a" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="8" markerHeight="8" orient="auto">
    <path d="M0,0 L10,5 L0,10 z" class="arrowhead"/></marker></defs>
  <rect class="node" x="10" y="20" width="110" height="40"/>
  <text x="65" y="45" text-anchor="middle">Tokenizer</text>
  <path class="edge" d="M120,40 L200,40" marker-end="url(#a)"/>
  <rect class="node node-accent" x="200" y="20" width="110" height="40"/>
  <text x="255" y="45" text-anchor="middle" class="on-accent">Scheduler</text>
</svg>
```

Every attribute references a design-system token; swap the theme and the diagram re-themes with it. Note the `class="on-accent"` on the label inside the terracotta node — without it, the label inherits the default dark text color and becomes unreadable against the accent fill. **This is the single most common SVG-theming bug. Pattern-match this example.**

**Common demo types**:

| Demo Type | Good For | Implementation |
|-----------|----------|---------------|
| **Architecture diagram** | System overview, component relationships, process boundaries | Inline SVG with labeled `<rect>` nodes + `<path>` connections |
| **Sequence diagram** | Request lifecycle, protocol exchanges, who-calls-whom over time | Inline SVG: vertical lanes per actor, horizontal arrows with labels |
| **State diagram** | FSMs, lifecycle transitions, request status | Inline SVG: `<circle>` states + directed edges; click to walk the machine |
| **Timeline** | Parallel/serial execution, pipeline stages | Horizontal SVG or CSS bars with time axis |
| **Grid/Memory** | Memory allocation, paging, caching | CSS grid of colored cells (or SVG if spatial relationships matter) |
| **Flow animation** | Request lifecycle, data pipelines | SVG nodes + arrows; highlight nodes sequentially on play |
| **Tree** | Caches, routing, decision trees, radix structures | SVG preferred for geometric layout; nested divs acceptable for simple trees |
| **Bar chart** | Probability distributions, comparisons | Animated CSS bars or inline SVG `<rect>` |
| **Step-through** | Algorithms, state machines | "Next Step" button advancing a pointer through an SVG diagram |
| **Simulation** | Queues, schedulers, batching | Auto-running SVG scene with live stats |
| **Calculator** | Memory budgets, performance | Sliders + computed output fields |
| **Side-by-side** | Before/after, with/without optimization | Two SVG panels animating simultaneously |
| **Tokenizer** | Text processing | Text input → colored spans output |

**Default to SVG** for rows marked "inline SVG" unless the concept is genuinely textual (Tokenizer) or tabular (Calculator). Reach for `<div>` only when SVG would be overkill.

### Step 4.2b: Reader Components

Three reader-facing UI patterns every tutorial ships. The shell (Step 4.1)
emits the CSS + the one global script; every chapter's draft expands into
these DOM shapes. All three inherit design-system tokens — never hardcode.

**1. Glossary tooltip** — hover any technical term → plain-English definition.

Shell CSS (emit once):
```css
.glossary { border-bottom: 1px dashed var(--border-strong); cursor: help; position: relative; }
.glossary[data-def]:hover::after,
.glossary[data-def]:focus-visible::after {
  content: attr(data-def);
  position: absolute; bottom: calc(100% + 6px); left: 0;
  max-width: 320px; padding: 8px 10px;
  background: var(--bg-tertiary); color: var(--text-bright);
  border: 1px solid var(--border); border-radius: 6px;
  box-shadow: var(--shadow-md);
  font-size: 13px; font-weight: 400; line-height: 1.4;
  white-space: normal; z-index: 50;
}
.glossary:focus-visible { outline: 2px solid var(--accent); outline-offset: 2px; }
```

Chapter-side emission:
```html
A <span class="glossary" tabindex="0" data-def="Table of previously computed attention values the model reuses instead of recomputing each token.">KV cache</span> lets decoding skip work it already did.
```

Every `data-def` value must come from the Glossary table in the Consistency
Bible — never paraphrased in place. Tooltip text is plain, not HTML.

**2. Quick Check quiz** — scenario, 2–4 choices, explanation per choice.

Shell CSS + JS (emit once):
```css
.quiz { border: 1px solid var(--border); border-radius: 10px;
  padding: 18px 20px; margin: 1.5rem 0; background: var(--bg-secondary); }
.quiz > .q-prompt { font-weight: 600; margin-bottom: 12px; color: var(--text-bright); }
.quiz .q-choice { display: block; width: 100%; text-align: left;
  background: var(--bg); border: 1px solid var(--border); border-radius: 8px;
  padding: 10px 14px; margin: 6px 0; cursor: pointer;
  font: inherit; color: var(--text); transition: background 120ms, border-color 120ms; }
.quiz .q-choice:hover { border-color: var(--border-strong); }
.quiz .q-choice.correct { background: color-mix(in srgb, var(--green) 14%, var(--bg));
  border-color: var(--green); }
.quiz .q-choice.wrong   { background: color-mix(in srgb, var(--red) 14%, var(--bg));
  border-color: var(--red); opacity: 0.85; }
.quiz .q-explain { margin-top: 10px; padding: 10px 12px; border-radius: 6px;
  background: var(--bg-tertiary); font-size: 14px; line-height: 1.5; display: none; }
.quiz.answered .q-explain { display: block; }
```
```js
document.addEventListener('click', (e) => {
  const btn = e.target.closest('.quiz .q-choice'); if (!btn) return;
  const quiz = btn.closest('.quiz'); if (quiz.classList.contains('answered')) return;
  const correct = btn.dataset.correct === 'true';
  btn.classList.add(correct ? 'correct' : 'wrong');
  quiz.classList.add('answered');
  const expl = quiz.querySelector(`.q-explain[data-for="${btn.dataset.id}"]`);
  if (expl) expl.style.display = 'block';
});
```

Chapter-side emission:
```html
<aside class="quiz">
  <div class="q-prompt">A new request arrives while the batch is mid-decode. Which file decides whether it waits for the next step or gets chunked in immediately?</div>
  <button class="q-choice" data-id="a" data-correct="false">scheduler.py</button>
  <button class="q-choice" data-id="b" data-correct="true">schedule_batch.py</button>
  <button class="q-choice" data-id="c" data-correct="false">model_runner.py</button>
  <div class="q-explain" data-for="a">Close — `scheduler.py` *orchestrates* the loop but doesn't admit individual requests.</div>
  <div class="q-explain" data-for="b"><code>schedule_batch.py</code> — `add_new_request()` at L88. Chunked prefill (Ch 6) lets the new request ride along instead of waiting.</div>
  <div class="q-explain" data-for="c">`model_runner.py` runs the forward pass; admission happens upstream.</div>
</aside>
```

One quiz per chapter, maximum. Omit if the chapter has no genuine
decision-point scenario — padding with fake-application quizzes is worse
than no quiz.

### Step 4.3: Visual Consistency Gate

Dispatch **1 subagent** (type: `general-purpose`) to run mechanical, invariant-shaped checks on the assembled HTML. These are grep-level, deterministic, and cheap — exactly the kind of check that catches regressions every run if skipped.

**Palette closure**
- Extract every hex color and `rgb()` literal appearing in the emitted HTML (excluding any verbatim source-code snippets in `.source-code` blocks).
- Every extracted color must resolve to a DESIGN.md token (either the literal value, or a `var(--token)` reference).
- Any color outside the palette is a violation — fix by replacing with the nearest token.

**SVG invariants** — for every `<svg class="diagram">`:
- No `<foreignObject>`, no `<image xlink:href="data:">`, no `<script>`, no external `href=`.
- Every `<text>` inside a `.node-accent`, `.info`, `.success`, `.warning`, or `.error` filled node must carry `class="on-accent"` (or equivalent inverse-text class) — otherwise text contrast fails.
- Every `fill="#..."` or `stroke="#..."` on a classed element must be accompanied by `style="fill:...;"` inline, OR the CSS rule must use `:where()` — bare `fill="..."` attributes will be overridden by the scaffold's class rules. (See Known Traps.)
- At most one `.node-accent` per diagram (accent is for emphasis, not decoration).

**Contrast spot check**
- For every distinct (text-color, background-color) pair used in `<text>` elements, compute WCAG contrast ratio. Must be ≥ 4.5:1. Flag violations.
- For every syntax-highlight class (`.kw`, `.fn`, `.st`, `.cm`, `.an`, `.num`, `.op`, `.cls`, `.dec`) rendered on `--code-bg`: compute contrast ratio. Must be ≥ 4.5:1. This catches the classic regression of pasting dark-mode syntax colors onto a light code surface.

**Determinism audit**
- `grep -n "Math.random" $OUTPUT` → must return 0 hits outside `.source-code` blocks.

**Dark-mode gate**
- If DESIGN.md has no dark palette, `grep -n "prefers-color-scheme" $OUTPUT` must return 0 hits.
- If DESIGN.md has a dark palette, every token referenced inside the dark media query must exist in the palette.

**Diagram layout gate** — for every `<svg class="diagram">`:
- For every `<rect class="node">` that contains a `<text>` label, compute
  `expected_width = (longest_line_chars × 7) + 32`. If the node's `width`
  attribute is less than `expected_width`, the box clips its label → fail
  and expand the node.
- For every `<text>` with `font-size` attribute present, assert value ≥ 11.
  Lower values mean someone shrunk the label to fit; fix by growing the
  box, not shrinking the text.
- For every `<text>` element whose y-coordinate places it on or crossing
  a `<path class="edge">`, assert a `<rect class="text-halo">` appears
  immediately before the `<text>` in source order (same approximate
  coords). Missing halo → fail.
- `viewBox` padding: after computing the tightest bounding box of every
  `<rect>` and `<text>`, assert the viewBox extends at least 24px beyond
  on all four sides. Tight viewBoxes clip outer labels.

**Chapter 1 overview gate**
- The first `<section class="chapter">` must contain at least one
  `<svg class="diagram">` with **≥ 3 distinct labeled nodes** (count
  `<text>` elements that aren't edge labels or axis ticks).
- Every node label must resolve to exactly one of:
  (a) a later chapter's title or subtitle in `chapter_plan.md`, or
  (b) an entry in an explicit "Not covered" list inside Ch 1 itself.
  Orphan labels (matching neither) are a fail — either a chapter is
  missing, or the component shouldn't be drawn.
- Ch 1 must contain **no `<pre>` code blocks longer than 12 lines** and
  **no `<div class="interactive">`** other than the worked-example
  animation (if any). This catches the most common failure mode: Ch 1
  drifting into "let me also explain the scheduler real quick".

**Reader-component gates**
- **Glossary closure**: every `<span class="glossary" data-def="...">` term
  must appear in the Consistency Bible Glossary, and its `data-def` must
  match the Bible's definition byte-for-byte (no paraphrases). Grep extracts
  the term text + `data-def`, diffs against `consistency.md` glossary.
- **References are hyperlinks**: every `<li>` inside `.further-reading` must
  contain at least one `<a href="http...">` element whose `href` starts with
  `https://` (or `http://` — `arxiv.org`, `doi.org`, project homepages, canonical
  blog posts). Bare text paper citations fail the gate. Additionally: the
  literal strings `[arXiv link]`, `<arxiv>`, `<url>`, or `(TBD)` must not
  appear anywhere in the output — they are template placeholders the writer
  forgot to fill in.
- **Quiz completeness**: every `.quiz` must have exactly one `data-correct="true"`
  choice and a `.q-explain[data-for=...]` entry per choice. Missing explanations
  are a fail.
- **No wall-of-text sections**: for each `<section class="chapter">`, walk
  the DOM within each `###`-level subsection and flag any run of more than
  ~8 consecutive `<p>` elements without an interrupting visual
  (`<pre>`, `<figure>`, `<aside class="quiz">`,
  `svg.diagram`, `<div class="interactive">`, `<ul>`/`<ol>`, callout div).
  Report violations as chapter ID + subsection heading.

**Bounded-scroll gate (UI invariant)**
Scroll must happen **inside** the content column, not at the page level. Checks:
- `body { overflow: hidden }` or equivalent — the window itself never scrolls.
- The element containing chapters (`.content` or equivalent) has `height: 100vh` and `overflow-y: auto`.
- The sidebar is fixed (`position: fixed; height: 100vh`) — it must not scroll out of view when the content scrolls.
- On chapter switch, the content column resets to `scrollTop = 0` in JS.
Report as `SCROLL: ok` or `SCROLL: fail — body scrolls at page level, content column not bounded`.

Reviewer returns a ≤120-word report:
```
PALETTE: 0 violations
SVG INVARIANTS: fixed 2 (added on-accent to ch03, removed <foreignObject> from ch07)
CONTRAST: 1 failure → fixed (label on warning node)
DETERMINISM: ok
DARK-MODE: ok (no override emitted; Claude brand has no dark palette)
```

Fixes are applied directly to the HTML file. If a fix requires re-rendering a chapter's demo (structural change), re-dispatch that chapter's Step 4.2 agent with the failure report.

### Step 4.4: Polish

- Verify every chapter is reachable via sidebar navigation
- Verify every demo has a reset button and no broken states
- Verify no external URLs are fetched (fully offline)
- Verify mobile layout works (sidebar collapses, demos resize)
- Add `<meta>` tags for social sharing (title, description)

**Output**: Write to `$OUTPUT` (default: `<project_name>_tutorial.html` — see Parse Arguments for derivation rules)

---

## Key Lessons

1. **Concept order is everything.** If chapter 5 uses a term from chapter 8, the reader is lost. The dependency graph in Step 1.2 is a first pass; the **binding** order is locked in Step 1.4 Chapter Planning after deep dives, because Step 1.3 reveals merges/splits that 1.2 can't see.

1b. **Lock in chapter structure before drafting, never during.** Changing chapter boundaries after Phase 2 begins is the most expensive mistake in the pipeline — it cascades into re-splits of prose, re-runs of per-chapter reviews, re-writes of demos, and re-assembly of HTML. Step 1.4 is the single gate. If a downstream step reveals the plan is wrong, re-dispatch Step 1.4 to produce a new plan; do not ad-hoc mid-draft edits.

2. **One interactive demo per chapter, minimum.** Text-only chapters lose engagement.

3. **Never have a single agent write the entire HTML file.** It will exceed context limits. Chapters → per-file disk writes in Step 2.2 → shell agent in Step 4.0 → orchestrator stitches with Edit inserts. See Step 4.0 for the exact two-stage assembly protocol.

4. **Verify citations early (Step 1.5), not late.** A broken `file.py:L42-L78` reference caught in research notes costs one edit; the same error caught in Round 1 review costs a re-render of every chapter that references it.

4b. **Two-pass drafting: raw content first, prose second.** Reviewing prose is expensive because reviewers must distinguish "wrong fact" from "awkward sentence". Reviewing bullets is trivial. Step 2.2 writes structured raw content (bullets/facts/snippets), Step 2.3 catches every factual and consistency issue in that cheap form, Step 2.4 turns approved raw into prose, Step 2.5 only checks flow. Per-chapter reviewers fire the moment each file lands — don't batch at the end. Bulk review (Phase 3 Round 1) then shrinks to cross-chapter concerns only: terminology drift, redundancy, broken cross-refs.

5. **Split research by architectural layer, not by chapter count.** Concepts in the same layer share vocabulary; concepts across layers don't. Thirds-splitting makes agents research each other's dependencies in parallel.

6. **Interactive demos must have reset buttons AND seeded RNG.** Reset alone isn't enough if `Math.random()` reshuffles the state on every click — two users comparing notes need to see identical numbers. Seed once in the shell, reseed on reset.

7. **Stats counters make abstract concepts click.** "Cache hit rate: 73%" beats "the cache frequently avoids recomputation."

8. **Side-by-side comparison is the most powerful demo type.** "Without X" vs "With X" simultaneously gives immediate intuition.

7. **Real source code builds trust that pseudocode cannot.** For every core mechanism, include the actual implementation trimmed to 10-40 lines with inline annotations. Source snippets also catch the writer — if you can't find a clean snippet, you don't understand the mechanism well enough.

8. **Source code snippets must be visually distinct from pseudocode.** Use a different border, a `SOURCE CODE` tag, and the file path in the header.

9. **The final HTML must be a single file with zero external dependencies.** Works offline, loads instantly, hostable anywhere.

10. **Apply a design system, don't invent one.** Use `--design <brand>` to pull tokens from getdesign.md (68+ brands available). The Claude design is the built-in default. All colors, typography, shadows, and border-radius come from the design system — CSS custom properties make swapping trivial.

11. **Responsive layout is not optional.** Many readers open on mobile after someone shares the link.

12. **Explain the technique, not just its usage.** If a chapter introduces a radix tree, explain what a radix tree IS before explaining how the codebase uses one. If a chapter uses speculative decoding, explain why memory-bandwidth-bound inference makes it work. The reader who doesn't know the underlying concept will bounce — the reader who already knows will skim past the explanation in 5 seconds. Always err on the side of explaining.

13. **Every design decision deserves a "why not the alternative?"** The most valuable insight in any tutorial is not "the system uses X" but "the system uses X instead of Y because of Z." This is the difference between documentation (what) and education (why). Dedicate a comparison table or paragraph to the rejected alternative in every chapter where a non-obvious choice was made.

---

## Adaptation by Project Type

| Project Type | Concept Focus | Best Demo Types |
|---|---|---|
| **Web framework** | Request lifecycle, middleware, routing, ORM | Flow animation, step-through |
| **Database** | Storage engine, query planning, indexing, transactions | Tree, memory grid, timeline |
| **Compiler** | Lexing, parsing, AST, type-checking, codegen | Tokenizer, tree, step-through |
| **ML system** | Tensors, GPU, model arch, caching, batching | Grid, timeline, simulation |
| **Distributed system** | Consensus, replication, partitioning | Simulation, flow animation |
| **CLI tool** | Parsing, plugins, execution model | Step-through, flow animation |
| **OS kernel** | Processes, memory, filesystem, scheduling | Memory grid, timeline |

## Adaptation by Audience Level

| Level | First Principles | Code References | Analogies |
|---|---|---|---|
| **Beginner** | Explain everything | Pseudocode only | Heavy use |
| **Intermediate** | Explain domain concepts, skip CS basics | Real code + pseudocode | For novel concepts only |
| **Expert** | Skip basics, focus on design decisions | Full code with file:line | None, use precise terminology |

---

## Appendix A: Built-in Claude Design Tokens (Default)

When `--design claude` or no design flag is specified, use these tokens. This is the Claude (Anthropic) design system — warm, editorial, literary.
# Design System Inspired by Claude (Anthropic)

## 1. Visual Theme & Atmosphere

Claude's interface is a literary salon reimagined as a product page — warm, unhurried, and quietly intellectual. The entire experience is built on a parchment-toned canvas (`#f5f4ed`) that deliberately evokes the feeling of high-quality paper rather than a digital surface. Where most AI product pages lean into cold, futuristic aesthetics, Claude's design radiates human warmth, as if the AI itself has good taste in interior design.

The signature move is the custom Anthropic Serif typeface — a medium-weight serif with generous proportions that gives every headline the gravitas of a book title. Combined with organic, hand-drawn-feeling illustrations in terracotta (`#c96442`), black, and muted green, the visual language says "thoughtful companion" rather than "powerful tool." The serif headlines breathe at tight-but-comfortable line-heights (1.10–1.30), creating a cadence that feels more like reading an essay than scanning a product page.

What makes Claude's design truly distinctive is its warm neutral palette. Every gray has a yellow-brown undertone (`#5e5d59`, `#87867f`, `#4d4c48`) — there are no cool blue-grays anywhere. Borders are cream-tinted (`#f0eee6`, `#e8e6dc`), shadows use warm transparent blacks, and even the darkest surfaces (`#141413`, `#30302e`) carry a barely perceptible olive warmth. This chromatic consistency creates a space that feels lived-in and trustworthy.

**Key Characteristics:**
- Warm parchment canvas (`#f5f4ed`) evoking premium paper, not screens
- Custom Anthropic type family: Serif for headlines, Sans for UI, Mono for code
- Terracotta brand accent (`#c96442`) — warm, earthy, deliberately un-tech
- Exclusively warm-toned neutrals — every gray has a yellow-brown undertone
- Organic, editorial illustrations replacing typical tech iconography
- Ring-based shadow system (`0px 0px 0px 1px`) creating border-like depth without visible borders
- Magazine-like pacing with generous section spacing and serif-driven hierarchy

## 2. Color Palette & Roles

### Primary
- **Anthropic Near Black** (`#141413`): The primary text color and dark-theme surface — not pure black but a warm, almost olive-tinted dark that's gentler on the eyes. The warmest "black" in any major tech brand.
- **Terracotta Brand** (`#c96442`): The core brand color — a burnt orange-brown used for primary CTA buttons, brand moments, and the signature accent. Deliberately earthy and un-tech.
- **Coral Accent** (`#d97757`): A lighter, warmer variant of the brand color used for text accents, links on dark surfaces, and secondary emphasis.

### Secondary & Accent
- **Error Crimson** (`#b53333`): A deep, warm red for error states — serious without being alarming.
- **Focus Blue** (`#3898ec`): Standard blue for input focus rings — the only cool color in the entire system, used purely for accessibility.

### Surface & Background
- **Parchment** (`#f5f4ed`): The primary page background — a warm cream with a yellow-green tint that feels like aged paper. The emotional foundation of the entire design.
- **Ivory** (`#faf9f5`): The lightest surface — used for cards and elevated containers on the Parchment background. Barely distinguishable but creates subtle layering.
- **Pure White** (`#ffffff`): Reserved for specific button surfaces and maximum-contrast elements.
- **Warm Sand** (`#e8e6dc`): Button backgrounds and prominent interactive surfaces — a noticeably warm light gray.
- **Dark Surface** (`#30302e`): Dark-theme containers, nav borders, and elevated dark elements — warm charcoal.
- **Deep Dark** (`#141413`): Dark-theme page background and primary dark surface.

### Neutrals & Text
- **Charcoal Warm** (`#4d4c48`): Button text on light warm surfaces — the go-to dark-on-light text.
- **Olive Gray** (`#5e5d59`): Secondary body text — a distinctly warm medium-dark gray.
- **Stone Gray** (`#87867f`): Tertiary text, footnotes, and de-emphasized metadata.
- **Dark Warm** (`#3d3d3a`): Dark text links and emphasized secondary text.
- **Warm Silver** (`#b0aea5`): Text on dark surfaces — a warm, parchment-tinted light gray.

### Semantic & Accent
- **Border Cream** (`#f0eee6`): Standard light-theme border — barely visible warm cream, creating the gentlest possible containment.
- **Border Warm** (`#e8e6dc`): Prominent borders, section dividers, and emphasized containment on light surfaces.
- **Border Dark** (`#30302e`): Standard border on dark surfaces — maintains the warm tone.
- **Ring Warm** (`#d1cfc5`): Shadow ring color for button hover/focus states.
- **Ring Subtle** (`#dedc01`): Secondary ring variant for lighter interactive surfaces.
- **Ring Deep** (`#c2c0b6`): Deeper ring for active/pressed states.

### Gradient System
- Claude's design is **gradient-free** in the traditional sense. Depth and visual richness come from the interplay of warm surface tones, organic illustrations, and light/dark section alternation. The warm palette itself creates a "gradient" effect as the eye moves through cream → sand → stone → charcoal → black sections.

## 3. Typography Rules

### Font Family
- **Headline**: `Anthropic Serif`, with fallback: `Georgia`
- **Body / UI**: `Anthropic Sans`, with fallback: `Arial`
- **Code**: `Anthropic Mono`, with fallback: `Arial`

*Note: These are custom typefaces. For external implementations, Georgia serves as the serif substitute and system-ui/Inter as the sans substitute.*

### Hierarchy

| Role | Font | Size | Weight | Line Height | Letter Spacing | Notes |
|------|------|------|--------|-------------|----------------|-------|
| Display / Hero | Anthropic Serif | 64px (4rem) | 500 | 1.10 (tight) | normal | Maximum impact, book-title presence |
| Section Heading | Anthropic Serif | 52px (3.25rem) | 500 | 1.20 (tight) | normal | Feature section anchors |
| Sub-heading Large | Anthropic Serif | 36–36.8px (~2.3rem) | 500 | 1.30 | normal | Secondary section markers |
| Sub-heading | Anthropic Serif | 32px (2rem) | 500 | 1.10 (tight) | normal | Card titles, feature names |
| Sub-heading Small | Anthropic Serif | 25–25.6px (~1.6rem) | 500 | 1.20 | normal | Smaller section titles |
| Feature Title | Anthropic Serif | 20.8px (1.3rem) | 500 | 1.20 | normal | Small feature headings |
| Body Serif | Anthropic Serif | 17px (1.06rem) | 400 | 1.60 (relaxed) | normal | Serif body text (editorial passages) |
| Body Large | Anthropic Sans | 20px (1.25rem) | 400 | 1.60 (relaxed) | normal | Intro paragraphs |
| Body / Nav | Anthropic Sans | 17px (1.06rem) | 400–500 | 1.00–1.60 | normal | Navigation links, UI text |
| Body Standard | Anthropic Sans | 16px (1rem) | 400–500 | 1.25–1.60 | normal | Standard body, button text |
| Body Small | Anthropic Sans | 15px (0.94rem) | 400–500 | 1.00–1.60 | normal | Compact body text |
| Caption | Anthropic Sans | 14px (0.88rem) | 400 | 1.43 | normal | Metadata, descriptions |
| Label | Anthropic Sans | 12px (0.75rem) | 400–500 | 1.25–1.60 | 0.12px | Badges, small labels |
| Overline | Anthropic Sans | 10px (0.63rem) | 400 | 1.60 | 0.5px | Uppercase overline labels |
| Micro | Anthropic Sans | 9.6px (0.6rem) | 400 | 1.60 | 0.096px | Smallest text |
| Code | Anthropic Mono | 15px (0.94rem) | 400 | 1.60 | -0.32px | Inline code, terminal |

### Principles
- **Serif for authority, sans for utility**: Anthropic Serif carries all headline content with medium weight (500), giving every heading the gravitas of a published title. Anthropic Sans handles all functional UI text — buttons, labels, navigation — with quiet efficiency.
- **Single weight for serifs**: All Anthropic Serif headings use weight 500 — no bold, no light. This creates a consistent "voice" across all headline sizes, as if the same author wrote every heading.
- **Relaxed body line-height**: Most body text uses 1.60 line-height — significantly more generous than typical tech sites (1.4–1.5). This creates a reading experience closer to a book than a dashboard.
- **Tight-but-not-compressed headings**: Line-heights of 1.10–1.30 for headings are tight but never claustrophobic. The serif letterforms need breathing room that sans-serif fonts don't.
- **Micro letter-spacing on labels**: Small sans text (12px and below) uses deliberate letter-spacing (0.12px–0.5px) to maintain readability at tiny sizes.

## 4. Component Stylings

### Buttons

**Warm Sand (Secondary)**
- Background: Warm Sand (`#e8e6dc`)
- Text: Charcoal Warm (`#4d4c48`)
- Padding: 0px 12px 0px 8px (asymmetric — icon-first layout)
- Radius: comfortably rounded (8px)
- Shadow: ring-based (`#e8e6dc 0px 0px 0px 0px, #d1cfc5 0px 0px 0px 1px`)
- The workhorse button — warm, unassuming, clearly interactive

**White Surface**
- Background: Pure White (`#ffffff`)
- Text: Anthropic Near Black (`#141413`)
- Padding: 8px 16px 8px 12px
- Radius: generously rounded (12px)
- Hover: shifts to secondary background color
- Clean, elevated button for light surfaces

**Dark Charcoal**
- Background: Dark Surface (`#30302e`)
- Text: Ivory (`#faf9f5`)
- Padding: 0px 12px 0px 8px
- Radius: comfortably rounded (8px)
- Shadow: ring-based (`#30302e 0px 0px 0px 0px, ring 0px 0px 0px 1px`)
- The inverted variant for dark-on-light emphasis

**Brand Terracotta**
- Background: Terracotta Brand (`#c96442`)
- Text: Ivory (`#faf9f5`)
- Radius: 8–12px
- Shadow: ring-based (`#c96442 0px 0px 0px 0px, #c96442 0px 0px 0px 1px`)
- The primary CTA — the only button with chromatic color

**Dark Primary**
- Background: Anthropic Near Black (`#141413`)
- Text: Warm Silver (`#b0aea5`)
- Padding: 9.6px 16.8px
- Radius: generously rounded (12px)
- Border: thin solid Dark Surface (`1px solid #30302e`)
- Used on dark theme surfaces

### Cards & Containers
- Background: Ivory (`#faf9f5`) or Pure White (`#ffffff`) on light surfaces; Dark Surface (`#30302e`) on dark
- Border: thin solid Border Cream (`1px solid #f0eee6`) on light; `1px solid #30302e` on dark
- Radius: comfortably rounded (8px) for standard cards; generously rounded (16px) for featured; very rounded (32px) for hero containers and embedded media
- Shadow: whisper-soft (`rgba(0,0,0,0.05) 0px 4px 24px`) for elevated content
- Ring shadow: `0px 0px 0px 1px` patterns for interactive card states
- Section borders: `1px 0px 0px` (top-only) for list item separators

### Inputs & Forms
- Text: Anthropic Near Black (`#141413`)
- Padding: 1.6px 12px (very compact vertical)
- Border: standard warm borders
- Focus: ring with Focus Blue (`#3898ec`) border-color — the only cool color moment
- Radius: generously rounded (12px)

### Navigation
- Sticky top nav with warm background
- Logo: Claude wordmark in Anthropic Near Black
- Links: mix of Near Black (`#141413`), Olive Gray (`#5e5d59`), and Dark Warm (`#3d3d3a`)
- Nav border: `1px solid #30302e` (dark) or `1px solid #f0eee6` (light)
- CTA: Terracotta Brand button or White Surface button
- Hover: text shifts to foreground-primary, no decoration

### Image Treatment
- Product screenshots showing the Claude chat interface
- Generous border-radius on media (16–32px)
- Embedded video players with rounded corners
- Dark UI screenshots provide contrast against warm light canvas
- Organic, hand-drawn illustrations for conceptual sections

### Distinctive Components

**Model Comparison Cards**
- Opus 4.5, Sonnet 4.5, Haiku 4.5 presented in a clean card grid
- Each model gets a bordered card with name, description, and capability badges
- Border Warm (`#e8e6dc`) separation between items

**Organic Illustrations**
- Hand-drawn-feeling vector illustrations in terracotta, black, and muted green
- Abstract, conceptual rather than literal product diagrams
- The primary visual personality — no other AI company uses this style

**Dark/Light Section Alternation**
- The page alternates between Parchment light and Near Black dark sections
- Creates a reading rhythm like chapters in a book
- Each section feels like a distinct environment

## 5. Layout Principles

### Spacing System
- Base unit: 8px
- Scale: 3px, 4px, 6px, 8px, 10px, 12px, 16px, 20px, 24px, 30px
- Button padding: asymmetric (0px 12px 0px 8px) or balanced (8px 16px)
- Card internal padding: approximately 24–32px
- Section vertical spacing: generous (estimated 80–120px between major sections)

### Grid & Container
- Max container width: approximately 1200px, centered
- Hero: centered with editorial layout
- Feature sections: single-column or 2–3 column card grids
- Model comparison: clean 3-column grid
- Full-width dark sections breaking the container for emphasis

### Whitespace Philosophy
- **Editorial pacing**: Each section breathes like a magazine spread — generous top/bottom margins create natural reading pauses.
- **Serif-driven rhythm**: The serif headings establish a literary cadence that demands more whitespace than sans-serif designs.
- **Content island approach**: Sections alternate between light and dark environments, creating distinct "rooms" for each message.

### Border Radius Scale
- Sharp (4px): Minimal inline elements
- Subtly rounded (6–7.5px): Small buttons, secondary interactive elements
- Comfortably rounded (8–8.5px): Standard buttons, cards, containers
- Generously rounded (12px): Primary buttons, input fields, nav elements
- Very rounded (16px): Featured containers, video players, tab lists
- Highly rounded (24px): Tag-like elements, highlighted containers
- Maximum rounded (32px): Hero containers, embedded media, large cards

## 6. Depth & Elevation

| Level | Treatment | Use |
|-------|-----------|-----|
| Flat (Level 0) | No shadow, no border | Parchment background, inline text |
| Contained (Level 1) | `1px solid #f0eee6` (light) or `1px solid #30302e` (dark) | Standard cards, sections |
| Ring (Level 2) | `0px 0px 0px 1px` ring shadows using warm grays | Interactive cards, buttons, hover states |
| Whisper (Level 3) | `rgba(0,0,0,0.05) 0px 4px 24px` | Elevated feature cards, product screenshots |
| Inset (Level 4) | `inset 0px 0px 0px 1px` at 15% opacity | Active/pressed button states |

**Shadow Philosophy**: Claude communicates depth through **warm-toned ring shadows** rather than traditional drop shadows. The signature `0px 0px 0px 1px` pattern creates a border-like halo that's softer than an actual border — it's a shadow pretending to be a border, or a border that's technically a shadow. When drop shadows do appear, they're extremely soft (0.05 opacity, 24px blur) — barely visible lifts that suggest floating rather than casting.

### Decorative Depth
- **Light/Dark alternation**: The most dramatic depth effect comes from alternating between Parchment (`#f5f4ed`) and Near Black (`#141413`) sections — entire sections shift elevation by changing the ambient light level.
- **Warm ring halos**: Button and card interactions use ring shadows that match the warm palette — never cool-toned or generic gray.

## 7. Do's and Don'ts

### Do
- Use Parchment (`#f5f4ed`) as the primary light background — the warm cream tone IS the Claude personality
- Use Anthropic Serif at weight 500 for all headlines — the single-weight consistency is intentional
- Use Terracotta Brand (`#c96442`) only for primary CTAs and the highest-signal brand moments
- Keep all neutrals warm-toned — every gray should have a yellow-brown undertone
- Use ring shadows (`0px 0px 0px 1px`) for interactive element states instead of drop shadows
- Maintain the editorial serif/sans hierarchy — serif for content headlines, sans for UI
- Use generous body line-height (1.60) for a literary reading experience
- Alternate between light and dark sections to create chapter-like page rhythm
- Apply generous border-radius (12–32px) for a soft, approachable feel

### Don't
- Don't use cool blue-grays anywhere — the palette is exclusively warm-toned
- Don't use bold (700+) weight on Anthropic Serif — weight 500 is the ceiling for serifs
- Don't introduce saturated colors beyond Terracotta — the palette is deliberately muted
- Don't use sharp corners (< 6px radius) on buttons or cards — softness is core to the identity
- Don't apply heavy drop shadows — depth comes from ring shadows and background color shifts
- Don't use pure white (`#ffffff`) as a page background — Parchment (`#f5f4ed`) or Ivory (`#faf9f5`) are always warmer
- Don't use geometric/tech-style illustrations — Claude's illustrations are organic and hand-drawn-feeling
- Don't reduce body line-height below 1.40 — the generous spacing supports the editorial personality
- Don't use monospace fonts for non-code content — Anthropic Mono is strictly for code
- Don't mix in sans-serif for headlines — the serif/sans split is the typographic identity

## 8. Responsive Behavior

### Breakpoints
| Name | Width | Key Changes |
|------|-------|-------------|
| Small Mobile | <479px | Minimum layout, stacked everything, compact typography |
| Mobile | 479–640px | Single column, hamburger nav, reduced heading sizes |
| Large Mobile | 640–767px | Slightly wider content area |
| Tablet | 768–991px | 2-column grids begin, condensed nav |
| Desktop | 992px+ | Full multi-column layout, expanded nav, maximum hero typography (64px) |

### Touch Targets
- Buttons use generous padding (8–16px vertical minimum)
- Navigation links adequately spaced for thumb navigation
- Card surfaces serve as large touch targets
- Minimum recommended: 44x44px

### Collapsing Strategy
- **Navigation**: Full horizontal nav collapses to hamburger on mobile
- **Feature sections**: Multi-column → stacked single column
- **Hero text**: 64px → 36px → ~25px progressive scaling
- **Model cards**: 3-column → stacked vertical
- **Section padding**: Reduces proportionally but maintains editorial rhythm
- **Illustrations**: Scale proportionally, maintain aspect ratios

### Image Behavior
- Product screenshots scale proportionally within rounded containers
- Illustrations maintain quality at all sizes
- Video embeds maintain 16:9 aspect ratio with rounded corners
- No art direction changes between breakpoints

## 9. Agent Prompt Guide

### Quick Color Reference
- Brand CTA: "Terracotta Brand (#c96442)"
- Page Background: "Parchment (#f5f4ed)"
- Card Surface: "Ivory (#faf9f5)"
- Primary Text: "Anthropic Near Black (#141413)"
- Secondary Text: "Olive Gray (#5e5d59)"
- Tertiary Text: "Stone Gray (#87867f)"
- Borders (light): "Border Cream (#f0eee6)"
- Dark Surface: "Dark Surface (#30302e)"

### Example Component Prompts
- "Create a hero section on Parchment (#f5f4ed) with a headline at 64px Anthropic Serif weight 500, line-height 1.10. Use Anthropic Near Black (#141413) text. Add a subtitle in Olive Gray (#5e5d59) at 20px Anthropic Sans with 1.60 line-height. Place a Terracotta Brand (#c96442) CTA button with Ivory text, 12px radius."
- "Design a feature card on Ivory (#faf9f5) with a 1px solid Border Cream (#f0eee6) border and comfortably rounded corners (8px). Title in Anthropic Serif at 25px weight 500, description in Olive Gray (#5e5d59) at 16px Anthropic Sans. Add a whisper shadow (rgba(0,0,0,0.05) 0px 4px 24px)."
- "Build a dark section on Anthropic Near Black (#141413) with Ivory (#faf9f5) headline text in Anthropic Serif at 52px weight 500. Use Warm Silver (#b0aea5) for body text. Borders in Dark Surface (#30302e)."
- "Create a button in Warm Sand (#e8e6dc) with Charcoal Warm (#4d4c48) text, 8px radius, and a ring shadow (0px 0px 0px 1px #d1cfc5). Padding: 0px 12px 0px 8px."
- "Design a model comparison grid with three cards on Ivory surfaces. Each card gets a Border Warm (#e8e6dc) top border, model name in Anthropic Serif at 25px, and description in Olive Gray at 15px Anthropic Sans."

### Iteration Guide
1. Focus on ONE component at a time
2. Reference specific color names — "use Olive Gray (#5e5d59)" not "make it gray"
3. Always specify warm-toned variants — no cool grays
4. Describe serif vs sans usage explicitly — "Anthropic Serif for the heading, Anthropic Sans for the label"
5. For shadows, use "ring shadow (0px 0px 0px 1px)" or "whisper shadow" — never generic "drop shadow"
6. Specify the warm background — "on Parchment (#f5f4ed)" or "on Near Black (#141413)"
7. Keep illustrations organic and conceptual — describe "hand-drawn-feeling" style
