# Storm Research — a Claude Code Agent Skill

Turn any topic into a **verified, multi-perspective research briefing delivered as a polished PDF**. Storm Research is a self-contained [Claude Code](https://claude.com/claude-code) Agent Skill. It runs a topic through five expert lenses, maps where they disagree, synthesizes a briefing, then adversarially peer-reviews itself and checks every citation against its primary source before rendering the final PDF.

The multi-perspective method is **inspired by Stanford's STORM** (Shao et al., *Assisting in Writing Wikipedia-like Articles From Scratch with Large Language Models*, NAACL 2024). The primary-source citation verification and the PDF report layer are additions on top of that idea — not part of the original STORM system.

## What it does

A 4-phase pipeline (heavier than a normal chat answer — that's the point):

1. **Five expert lenses** (parallel agents) — Practitioner, Academic, Skeptic, Economist, Historian each research the topic on the live web and return a sourced brief.
2. **Contradiction map** — where the lenses clash, what they all agree on (the load-bearing finding), and the blind spot none addressed.
3. **Synthesis** — a 60-second summary, five reliability-ranked findings, the hidden cross-lens connection, targeted actions, and a frontier question.
4. **Adversarial peer review + verification** — self-critique with 1–10 confidence scores, then independent agents verify every citation against its primary source. The report carries an honest banner: `X fabricated, Y corrected, Z demoted`.

The deliverable is a **single PDF**; the intermediate HTML is temporary and discarded.

## Install

Drop the `storm-research/` folder into a Claude Code skills directory:

```
~/.claude/skills/storm-research/          # user-level (all projects)
# or
<your-project>/.claude/skills/storm-research/   # project-level
```

The folder must contain `SKILL.md` and `report-template.html`.

## Use

In Claude Code:

```
/storm-research the best vendor for business people
```

or just ask in natural language — *"storm report on X"*, *"make me a briefing on X"*. Reports are written to `storm-reports/<topic-slug>-briefing.pdf` in your working directory.

## Requirements

- **Claude Code** (uses the built-in `Agent`, `Write`, and `Bash` tools + web search/fetch).
- **A Chromium browser** (Google Chrome or Microsoft Edge) installed — used in headless mode to render the PDF. Both ship by default on Windows/macOS.

No paid services, APIs, GitHub setup, or other skills required.

## Repository structure

- `storm-research/SKILL.md` — the skill: pipeline, agent prompts, and guardrails.
- `storm-research/report-template.html` — the PDF design template (modern editorial: dark hero, Space Grotesk / Inter / JetBrains Mono, violet + teal). Rendered to PDF; never shipped as HTML.
- `storm-reports/` — generated PDF briefings.

## Credit

- STORM (Stanford OVAL Lab): https://storm.genie.stanford.edu · https://github.com/stanford-oval/storm
- Paper: Shao et al., NAACL 2024.
