# teach-me-this-project

A Claude Code skill that generates a publish-ready interactive HTML tutorial for any software project — from first principles to implementation details.

## What it does

Point it at any codebase and it produces a single-file interactive website with:
- Chapter-by-chapter architecture walkthrough
- Real source code snippets (not pseudocode)
- Animated interactive demos per concept
- Sidebar navigation with progress tracking
- GitHub Pages–ready output

## Install

```bash
mkdir -p .claude/skills/teach-me-this-project
curl -o .claude/skills/teach-me-this-project/SKILL.md \
  https://raw.githubusercontent.com/odysa/teach-me-this-project/main/SKILL.md
```

Then restart Claude Code. The skill is available as `/teach-me-this-project`.

## Usage

```
/teach-me-this-project [project_path] [audience: beginner|intermediate|expert] [--design brand] [focus area]
```

**Examples:**
```
/teach-me-this-project
/teach-me-this-project ./my-repo expert
/teach-me-this-project https://github.com/org/repo beginner --design stripe
/teach-me-this-project . intermediate "focus on the scheduler"
```

## Design system

Ships with a `DESIGN.md` (Claude design system — warm parchment, terracotta accents). Drop it in your project root and the skill picks it up automatically. Override with `--design <brand>` to use any brand from [getdesign.md](https://getdesign.md/).

## Output

A single `tutorial.html` file (~2000–5000 lines), fully offline, no external dependencies.
