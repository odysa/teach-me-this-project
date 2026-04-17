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
- **OUTPUT**: Output filename (default: `tutorial.html`)

If the project is a GitHub URL and not cloned locally, clone it to `/tmp/<repo-name>` first.

### Design System Resolution

1. If `--design claude` or no `--design` flag: use the **Built-in Claude Design Tokens** in Appendix A below.
2. If `--design <other-brand>`: run `npx -y getdesign@latest add <brand>` in `/tmp`, read the resulting `DESIGN.md`, and extract color palette, typography, spacing, shadows, border-radius, and do's/don'ts. Apply those tokens instead of the Claude defaults in Phase 4.

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

Dispatch **3 parallel subagents** (type: `Explore`) to investigate the codebase, split by concept groups:

- **Agent A**: Concepts 1–N/3 (foundational)
- **Agent B**: Concepts N/3+1–2N/3 (intermediate)
- **Agent C**: Concepts 2N/3+1–N (advanced)

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

### Step 1.4: Fill Gaps

Review all Phase 1 outputs and identify:
- Missing links in the concept chain
- Prerequisite knowledge the audience might lack
- Cross-cutting concerns worth a chapter

Add 1-3 supplementary concepts if needed.

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

### Further Reading
Link to the original papers and references that introduced the techniques
used in this chapter. Not every chapter needs this — only chapters where the
technique has a clear research origin (most do in systems/ML codebases).

Format as a compact list:
- Paper title (Authors, Year) — one-line summary. [arXiv link]

### Interactive Demo
Description of the interactive visualization.

### Key Takeaway
One callout box summarizing the essential insight.
```

### Step 2.2: Write Chapters in Parallel

Dispatch **3 parallel subagents** to write chapter content:

- **Agent 1**: Introduction + Chapters 1–N/3 (~3000-5000 words)
- **Agent 2**: Chapters N/3+1–2N/3 (~3000-5000 words)
- **Agent 3**: Chapters 2N/3+1–N + Conclusion (~3000-5000 words)

Each agent receives the full concept dependency graph, the deep-dive traces for their concepts, the chapter template, and the target AUDIENCE level.

---

## Phase 3: Review

### Round 1: Technical Accuracy

Dispatch **2 parallel subagents** to cross-reference the draft against actual source code:
- Verify class/function names match the codebase
- Verify data structure descriptions are accurate
- Verify source code snippets are verbatim (re-read actual file and diff)
- Fix any drift, stale references, or renamed code

### Round 2: Pedagogy

Review the assembled draft sequentially:
- Flag undefined terms used before introduction
- Flag leaps in complexity without scaffolding
- Ensure "why should I care" motivation exists per chapter
- This review can be done inline (no need for a separate agent if the draft is solid)

---

## Phase 4: Interactive HTML Generation

### Step 4.1: HTML Architecture

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
- **Theme follows the design system, not the OS.** Use only the palette the design system defines. Do NOT add a `prefers-color-scheme: dark` override with invented dark colors — if the brand is fundamentally a light (or dark) design, honor that identity. Only emit a dark-mode block when the design system *explicitly* specifies a dark palette alongside the light one.
- All styling via CSS custom properties so the design system is a single block of overrides

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
- Header bar with `SOURCE CODE` tag + file path, on dark surface
- Inline annotations (`# ←`) in dimmer italic font, using the accent color
- Code blocks always use a dark surface background regardless of page theme

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

### Step 4.2: Build Interactive Demos

For each chapter, implement the interactive visualization. Every demo follows this pattern:

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

**Common demo types**:

| Demo Type | Good For | Implementation |
|-----------|----------|---------------|
| **Timeline** | Parallel/serial execution, pipeline stages | Horizontal colored bars |
| **Grid/Memory** | Memory allocation, caching | CSS grid of colored cells |
| **Flow animation** | Request lifecycle, data pipelines | Components + arrows |
| **Tree** | Caches, routing, decision trees | Nested divs or SVG |
| **Bar chart** | Probability distributions, comparisons | Animated CSS bars |
| **Step-through** | Algorithms, state machines | "Next Step" button |
| **Simulation** | Queues, schedulers, batching | Auto-running with stats |
| **Calculator** | Memory budgets, performance | Sliders + computed output |
| **Side-by-side** | Before/after, with/without optimization | Two panels animating |
| **Tokenizer** | Text processing | Text input → colored output |

### Step 4.3: Polish

- Verify every chapter is reachable via sidebar navigation
- Verify every demo has a reset button and no broken states
- Verify no external URLs are fetched (fully offline)
- Verify mobile layout works (sidebar collapses, demos resize)
- Add `<meta>` tags for social sharing (title, description)

**Output**: Write to `$OUTPUT` (default: `tutorial.html`)

---

## Key Lessons

1. **Concept order is everything.** If chapter 5 uses a term from chapter 8, the reader is lost. The dependency graph in Step 1.2 is the most important artifact.

2. **One interactive demo per chapter, minimum.** Text-only chapters lose engagement.

3. **Never have a single agent write the entire HTML file.** It will exceed context limits. Write prose chapters first, then generate the HTML shell + assemble.

4. **Interactive demos must have reset buttons.** Users WILL get the demo into a weird state.

5. **Stats counters make abstract concepts click.** "Cache hit rate: 73%" beats "the cache frequently avoids recomputation."

6. **Side-by-side comparison is the most powerful demo type.** "Without X" vs "With X" simultaneously gives immediate intuition.

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
