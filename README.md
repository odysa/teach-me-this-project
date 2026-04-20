<!--
  SCREENSHOTS NEEDED (drop into docs/images/):
  1. hero.gif             — 880×520, ~10s loop. Terminal runs /teach-me-this-project → browser opens tutorial.
  2. chapter-anatomy.png  — 880w. One chapter with callouts on citation, diagram, code, demo.
  3. ch1-architecture.png — 880w. Ch 1 page with full-system SVG + component-to-chapter map.
  4. demo.gif             — 780w, ~5s loop. User clicks a button, SVG animates.
  5. themes.png           — 880w, 3-panel. Same chapter in Claude / Stripe / Linear themes.
-->

<h1 align="center">teach-me-this-project</h1>

<p align="center"><b>Turn any GitHub repo into an interactive tutorial site — in one command.</b></p>

<p align="center">
Chapter-by-chapter walkthrough &nbsp;·&nbsp;
every claim cited to <code>file:line</code> &nbsp;·&nbsp;
animated diagrams &nbsp;·&nbsp;
one offline HTML file
</p>

<p align="center">
  <a href="https://odysa.github.io/project-tutorials/vllm.html"><b>→ See it on vLLM</b></a>
  &nbsp;·&nbsp;
  <a href="https://odysa.github.io/project-tutorials/">more examples</a>
  &nbsp;·&nbsp;
  <a href="#install">Install</a>
  &nbsp;·&nbsp;
  <a href="#how-its-different">Why it's different</a>
</p>

<p align="center">
  <img src="docs/images/hero.gif" width="880" alt="Running /teach-me-this-project generates a full interactive tutorial in one command.">
</p>

---

## Install

One line. Works across 18+ coding agents (Claude Code, Cursor, Copilot, Cline, …):

```bash
npx skills add odysa/teach-me-this-project
```

<details>
<summary>Native Claude Code install</summary>

```bash
# From inside Claude Code
/plugin marketplace add odysa/teach-me-this-project
/plugin install teach-me-this-project@teach-me-this-project
```

Uninstall: `/plugin uninstall teach-me-this-project@teach-me-this-project`.
</details>

## Use

```
/teach-me-this-project
```

That's it. Point it at the current repo, get back `<project>_tutorial.html`. Options when you want them:

```bash
/teach-me-this-project ./my-repo expert
/teach-me-this-project https://github.com/org/repo beginner --design stripe
/teach-me-this-project . intermediate "focus on the scheduler"
```

| Argument | Values | Default |
|---|---|---|
| `project_path` | local path or GitHub URL | current directory |
| `audience` | `beginner` · `intermediate` · `expert` | `intermediate` |
| `--design` | any brand from [getdesign.md](https://getdesign.md/) | `claude` |
| `focus area` | free-form string | full codebase |

## What you get

A single `<project>_tutorial.html` file. Zero dependencies. Works offline. `curl`-able. Drop into `gh-pages` and you're done.

<p align="center">
  <img src="docs/images/chapter-anatomy.png" width="880" alt="Anatomy of a chapter: cited source snippet, animated SVG diagram, interactive demo.">
  <br>
  <sub><i>Every chapter: cited source, diagram, demo.</i></sub>
</p>

| | |
|---|---|
| 📚 | **Chapter-by-chapter walkthrough** — each chapter assumes only what came before. |
| 🔗 | **Grep-verified citations** — every claim points to `file.py:L42-L78`. Citations are grep-checked against the source before the build finishes; a broken reference fails the pipeline. |
| 🗺 | **Architecture-first** — Chapter 1 is always a full-system diagram. No orphan boxes. |
| 🎬 | **Interactive demos** — animated, seeded, deterministic. Two readers see the same numbers. |
| 🎨 | **Your brand** — Claude default, or any design system from getdesign.md. |
| 🎚 | **Audience dial** — beginner, intermediate, or expert — same repo, three depths. |

<p align="center">
  <img src="docs/images/ch1-architecture.png" width="880" alt="Chapter 1 is always an architecture overview. Every component drawn maps to a later chapter — no orphan boxes.">
  <br>
  <sub><i>Chapter 1 is always an architecture overview — every component maps to a later chapter.</i></sub>
</p>

<p align="center">
  <img src="docs/images/demo.gif" width="780" alt="Every chapter ships at least one seeded, deterministic interactive demo.">
  <br>
  <sub><i>Interactive demos aren't decoration — they're the chapter's "aha" moment.</i></sub>
</p>

## <a name="how-its-different"></a>How this is different

| If you've been reaching for… | You get | What's missing |
|---|---|---|
| The repo's README | A high-level pitch | How it *actually* works, in order |
| Claude / ChatGPT chat | An improvised answer | Cited sources, structure, reproducibility |
| [DeepWiki](https://deepwiki.com) / hosted wikis | A chat Q&A | A linear walkthrough, an offline file, your brand |
| A static docs site (Mintlify, Docusaurus) | Polished pages | Interactivity, per-repo customization, zero setup |
| **`teach-me-this-project`** | A self-contained interactive tutorial site | — |

## Theme it to your brand

<p align="center">
  <img src="docs/images/themes.png" width="880" alt="The same tutorial, rendered in the Claude, Stripe, and Linear design systems.">
  <br>
  <sub><i>Same tutorial. Three brands. One flag: <code>--design &lt;brand&gt;</code>.</i></sub>
</p>

Any brand from [getdesign.md](https://getdesign.md/) works. The skill fetches the brand tokens and applies them to colors, typography, spacing, and shadows. No brand token is ever invented — if a design system has no dark palette, the tutorial ships light-only rather than guessing.

## How it works

The skill runs a multi-phase pipeline, most of it parallelized across subagents so the orchestrator's context stays clean:

1. **Investigate** — architecture survey → concept dependency graph → per-layer deep dives → citation verification.
2. **Plan** — lock the chapter list, Ch 1 always an architecture overview. No mid-flight restructuring.
3. **Draft (two-pass)** — raw bullets reviewed for *facts*, then a prose pass reviewed for *flow*. Bugs can't hide behind pretty writing.
4. **Assemble & gate** — shell + chapters stitched into one HTML, then grep-level invariant checks: no stray colors, no orphan citations, no dark-mode on light brands, every glossary term resolves.

Every pitfall found in past runs is encoded in a **Known Traps** list the skill reads before starting — so the same bug never ships twice.

## Publish to GitHub Pages

```bash
mkdir -p docs && cp *_tutorial.html docs/index.html
git add docs && git commit -m "Add tutorial" && git push
```

Settings → Pages → `main` branch, `/docs` folder. Live in a minute.

## Gallery

Every tutorial below was generated with this skill — click into any one and inspect the citations, demos, and diagrams yourself.

- **[Inside vLLM](https://odysa.github.io/project-tutorials/vllm.html)** — architecture and scheduling of the vLLM inference engine.
- **[SGLang Internals](https://odysa.github.io/project-tutorials/sglang.html)** — runtime and execution of the SGLang serving framework.
- **[nano-vLLM](https://odysa.github.io/project-tutorials/nano_vllm_tutorial.html)** — a minimal vLLM reimplementation, end to end.
- **[Mini-SGLang](https://odysa.github.io/project-tutorials/mini_sglang_tutorial.html)** — a beginner's tour of a modern LLM serving engine in twelve chapters.
- **[OpenClaw](https://odysa.github.io/project-tutorials/openclaw_tutorial.html)** — a multi-channel personal AI assistant: agentic loop, skills, sandboxed sub-agents.
- **[Hermes Agent](https://odysa.github.io/project-tutorials/hermes_agent_tutorial.html)** — NousResearch's agent: loop, tools, memory, gateway, extensions.
- **[Bub](https://odysa.github.io/project-tutorials/bub_tutorial.html)** — the Bub agent framework: architecture, primitives, runtime internals.
- **[pi-mono](https://odysa.github.io/project-tutorials/pi_mono_tutorial.html)** — a minimal coding agent: provider layer, agent loop, session compaction, TUI.

[**→ Full gallery**](https://odysa.github.io/project-tutorials/)

*Built one you're proud of? Open a PR to add it.*

## Feedback & contributing

Found a bug? Open an issue — if you can reproduce it, it gets added to the skill's **Known Traps** and the same bug never ships again.

## License

MIT
