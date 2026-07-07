---
name: storm-research
description: Use when someone asks to create, write, or generate a report, briefing, or research document on any topic, OR asks to run Storm Research / use the storm-research skill / run the STORM method, says "storm research this" / "storm report on X" / "make me a report on X" / "give me a briefing on X", or wants a multi-perspective, citation-verified research report. Produces a polished PDF (rendered from a self-contained HTML source). Runs a 4-phase pipeline: five expert lenses (Practitioner, Academic, Skeptic, Economist, Historian) -> contradiction map -> synthesized report -> adversarial peer review + primary-source verification. Default choice for any report request where multiple viewpoints and fact-checked claims matter; overkill for a simple one-line factual lookup.
argument-hint: "[topic to research]"
---

# Storm Research

## What this does

Turns one topic into a verified, multi-perspective briefing delivered as a **PDF only**. It simulates five expert lenses on the topic, maps where they contradict each other, synthesizes everything into a self-contained HTML render source, renders that to a print-ready PDF, then adversarially peer-reviews its own output and verifies every citation against its primary source before delivering. The deliverable is a single PDF with no blind spots and no unchecked claims; the intermediate HTML is temporary and discarded. The multi-perspective method is **inspired by Stanford's STORM** (Shao et al., NAACL 2024); the citation-verification and PDF layers here are additions on top of that idea, not part of the original STORM system.

Run the full pipeline end to end. Do not shortcut a phase. This is heavier than a quick web lookup; that is the point.

## Portability

This skill is self-contained. It depends only on built-in Claude Code tools (the `Agent` tool with the built-in `general-purpose` agent, `Write`, `Bash`, and web search/fetch used inside those agents) plus `report-template.html` in this same folder. The only external requirement is a **Chromium-based browser (Chrome or Microsoft Edge)** already installed on the machine — used in headless mode to render the HTML to PDF. Both ship by default on Windows/macOS. No paid services, APIs, or other skills are required. Drop the folder into any `.claude/skills/` directory and it works.

## Phase 0: Scope the topic

1. If `$ARGUMENTS` has the topic, use it. Otherwise ask what to research.
2. State your interpretation of the topic in one line and proceed. Only ask a clarifying question if the topic is genuinely ambiguous in a way that changes the research. Default to proceeding.
3. Identify the **reader's role** so the actionable section can target it. Infer it from the topic and any stated context; if unclear, ask in one line, or default to "a practitioner or decision-maker in this field."
4. Derive a kebab-case `topic-slug` from the topic for the filename.
5. Tell the user the pipeline is running (5 lenses, then verify). One line.

## Phase 1: Five expert lenses (parallel agents)

Spawn **five `general-purpose` agents in a single message** so they run concurrently. Each gets the SAME topic framing plus its own lens. Use these exact prompts, substituting `{TOPIC}` and a one-line `{TOPIC_FRAME}` (your Phase 0 interpretation):

**1. THE PRACTITIONER** — `You are THE PRACTITIONER for: {TOPIC} ({TOPIC_FRAME}). You work with this daily. Do real web research (prioritize recent sources, case studies, practitioner threads, operator data). Surface the GAP between what hands-on operators know and what academics/pundits miss, and the practical realities (workflow friction, what actually works, where it breaks) that get ignored. Return EXACTLY: 1) CORE POSITION in 2 sentences. 2) STRONGEST EVIDENCE, 3-5 bullets each with a concrete data point/case/named source + URL. 3) THE ONE THING only a practitioner would say. Cite real sources with URLs. Under 400 words.`

**2. THE ACADEMIC** — `You are THE ACADEMIC for: {TOPIC} ({TOPIC_FRAME}). You care about peer-reviewed evidence and effect sizes, not anecdotes. Do real web research (peer-reviewed studies, arXiv, university and research-institute reports, journals). Answer: what does the rigorous evidence ACTUALLY say vs popular belief, and where does it CONTRADICT the hype. Return EXACTLY: 1) CORE POSITION in 2 sentences. 2) STRONGEST EVIDENCE, 3-5 bullets each tied to a named study/report + URL with the actual finding/effect size. 3) THE ONE THING only an academic would say. Flag where evidence is thin or contested, and note peer-review status (published vs preprint). Under 400 words.`

**3. THE SKEPTIC** — `You are THE SKEPTIC for: {TOPIC} ({TOPIC_FRAME}). You think the mainstream view is overstated or wrong. Build the STRONGEST steelman bear case. Do real web research for backlash, failures, contradicting data, policy/regulatory changes, debunkings. Answer: the strongest counterargument, and what proponents conveniently ignore. Return EXACTLY: 1) CORE POSITION in 2 sentences. 2) STRONGEST EVIDENCE, 3-5 bullets each with a concrete source + URL. 3) THE ONE THING only a skeptic would say. Be rigorous, not contrarian for sport. Cite real sources with URLs. Under 400 words.`

**4. THE ECONOMIST** — `You are THE ECONOMIST for: {TOPIC} ({TOPIC_FRAME}). You follow the money. Do real web research for revenues, valuations, market size, funding flows, unit economics, incentives. Answer: who profits from the current narrative, and what financial incentives shape the research and hype. Return EXACTLY: 1) CORE POSITION in 2 sentences. 2) STRONGEST EVIDENCE, 3-5 bullets each with a real number (revenue/valuation/market size/funding) + named source + URL. 3) THE ONE THING only an economist would say (the follow-the-money insight). Cite real figures with URLs. Under 400 words.`

**5. THE HISTORIAN** — `You are THE HISTORIAN for: {TOPIC} ({TOPIC_FRAME}). You have seen disruption cycles before and look for patterns. Do real web research for genuine historical parallels (prior technologies, manias, market shifts). Answer: what parallels actually fit, and what we learn from how they played out (who won, who lost, what stabilized). Return EXACTLY: 1) CORE POSITION in 2 sentences. 2) STRONGEST EVIDENCE, 3-5 bullets each a specific historical case with dates/outcomes + a source URL. 3) THE ONE THING only a historian would say (the pattern no one else surfaces). Cite sources with URLs. Under 400 words.`

When all five return, post a 2-3 line note in chat: which way they converge, and the sharpest disagreement. Keep raw briefs out of chat (the agents already returned them).

## Phase 2: Map the contradictions

Working only from the five briefs, determine (do this inline, no agents):

1. **Direct conflicts** — where two or more lenses claim opposite things. Name the specific clashing claims, not just topics.
2. **Strongest vs weakest evidence** — which lens is best-supported (rank: peer-reviewed causal > official data > anecdote/analogy) and which is weakest, with why.
3. **The resolving question** — the single empirical question that would settle the biggest contradiction.
4. **Universal agreement** — what every lens confirms, even opponents. This is the likely-true load-bearing finding.
5. **The blind spot** — what NO lens addressed. This becomes the "missing 6th lens" and feeds the Frontier Question.

This map is not a separate deliverable. It is the raw material for the report's findings (supports/challenges), hidden connection, 6th-lens box, and frontier question.

## Phase 3: Synthesize the report (HTML render source)

The report is delivered as a **PDF only**. The HTML below is an internal render source, not a deliverable.

1. Read `report-template.html` in this skill folder. Clone it; do not rebuild the CSS. It is a modern design: dark hero, Space Grotesk / Inter / JetBrains Mono, violet + teal accent. Keep the `<style>` block and the Stanford STORM credit in the footer verbatim.
2. Fill every section. Mapping from the phases:
   - **60-second summary** — decision-maker-grade, nuance not headline. Lead with the settled fact, then the contested interpretation.
   - **5 key findings, ranked by reliability** — most important things now known, highest reliability first. Each carries a 1-10 confidence score (set in Phase 4) and Supported-by / Challenged-by chips drawn from the contradiction map.
   - **Hidden connection** — the non-obvious link from Phase 2 that only appears across all five lenses.
   - **Key assumption / missing 6th lens** — the blind spot from Phase 2, framed as the lens that could change the conclusions.
   - **Actionable insight** — 3-6 specific moves for the reader's role identified in Phase 0. Specific, not abstract.
   - **Claim safety guide** — assert / caveat / avoid, populated after Phase 4 verification.
   - **Frontier question** — the one question that would change everything.
   - **References** — every citation with a verification-status tag (set in Phase 4).
   - Each finding card takes TWO reliability classes: the outer `<div class="finding rl-...">` (colors the left bar) and the inner `<div class="rel ...">` — set both to `high | medhigh | medium | low` for that finding.
3. Write the working HTML to a **temp path** (the OS temp/scratch dir), e.g. `{temp}/{topic-slug}-briefing.html`. Do NOT write it into `storm-reports/` — that folder holds PDFs only. The PDF is rendered from this temp file in Output after Phase 4 corrections are applied. The HTML is never handed to the user.

## Phase 4: Adversarial peer review + verification (do not skip)

This is what separates Storm Research from a normal report. Run it before delivering.

**4a. Self-review (inline).** Score each of the 5 findings 1-10 for reliability and justify. Identify the weakest link and what would verify it. Run a bias check (which lens dominated the synthesis, what got underweighted). Name the missing 6th perspective. Assign an honest overall grade.

**4b. Verify every citation (parallel agents).** Spawn `general-purpose` agents in one message, one per distinct citation cluster (group related claims; ~4-6 agents). Each agent prompt:

`Independently verify a citation against its PRIMARY source. Be skeptical; do not trust secondary blog summaries. CLAIM: {claim + cited figure + named source}. Find the actual primary source. Confirm or correct: exact title/authors/venue/year/URL, the real figure or effect size as published, sample/method and any author-stated limits, and peer-review status (published vs preprint). For any contested claim, find the strongest credible counter-source. Return: VERDICT = CONFIRMED / PARTIALLY CONFIRMED (list corrections) / UNVERIFIED / FALSE, then the corrected one-line citation, then 2-4 bullets of specifics with the primary URL. Under 280 words.`

**4c. Apply corrections.** Edit the report:
- Fix any wrong figures, titles, dates, or mischaracterizations.
- Downgrade confidence scores where evidence turned out thin; demote preprints and contested claims into the "Contested signal" sidebar.
- Re-attribute single-survey or commissioned stats honestly.
- Fill the verification banner (`X fabricated, Y corrected, Z demoted`) and the per-citation status tags.
- Populate the claim safety guide from the verdicts.

## Output — PDF only

The deliverable is a single **PDF**. Only render it **after** Phase 4 corrections are written into the temp HTML, so the PDF is the verified v2.

1. **Render the temp HTML to PDF** with a headless Chromium browser, straight into the deliverable folder: `storm-reports/{topic-slug}-briefing.pdf` (create the folder if needed). **Use ABSOLUTE paths for both the temp input HTML and the output PDF** — Chrome/Edge `--print-to-pdf` silently fails on relative paths ("cannot find the path specified"). Use the first browser found:
   - **Windows** (Chrome, then Edge as fallback) — full absolute paths:
     ```
     & "C:\Program Files\Google\Chrome\Application\chrome.exe" --headless --disable-gpu --no-pdf-header-footer --user-data-dir="C:\temp\storm-chrome" --print-to-pdf="C:\full\path\storm-reports\{topic-slug}-briefing.pdf" "C:\temp\{topic-slug}-briefing.html"
     ```
     If Chrome is absent, swap in `C:\Program Files (x86)\Microsoft\Edge\Application\msedge.exe` with the same flags.
   - **macOS**: `"/Applications/Google Chrome.app/Contents/MacOS/Google Chrome" --headless --disable-gpu --no-pdf-header-footer --user-data-dir="/tmp/storm-chrome" --print-to-pdf="/abs/storm-reports/{topic-slug}-briefing.pdf" "/abs/tmp/{topic-slug}-briefing.html"`
   - **Linux**: `google-chrome --headless --disable-gpu --no-pdf-header-footer --user-data-dir="/tmp/storm-chrome" --print-to-pdf="/abs/.../briefing.pdf" "/abs/tmp/briefing.html"`
   - **Flag notes:** `--no-pdf-header-footer` removes the browser's date/URL margins; **`--user-data-dir=<a temp dir>` is required** — recent Chrome/Edge builds fail with `Missing headless user data directory` without it. The template's `@page` / `@media print` block supplies the dark hero, colored cards, and page breaks. Ignore noisy `GCM` / `sandboxed_unpacker` / `PHONE_REGISTRATION_ERROR` lines; the only line that matters is `NNNNNN bytes written to file ...`. Confirm the `.pdf` exists and is non-trivial (typically >100 KB with the embedded fonts/colors) before proceeding.
2. **Discard the temp HTML** (delete it, or just leave it in the OS temp dir). `storm-reports/` must contain the PDF only — never ship or reference the HTML.
3. **If no Chromium browser is available**, say so plainly and, as the only fallback, hand over the HTML so the work isn't lost — but note it is a fallback, not the intended deliverable.
4. **Final deliverable: `storm-reports/{topic-slug}-briefing.pdf`.** Open it with the platform's default opener: macOS `open <path>`, Linux `xdg-open <path>`, Windows PowerShell `Start-Process <path>`. If the OS is unclear, just give the path.
5. In chat, give: the PDF file path, the verification tally (`N/N checked, X fabricated, Y corrected, Z demoted`), the one universal finding, the frontier question, and the claim safety summary (what is safe to assert vs avoid). Keep it tight.

## Notes & guardrails

- **Real research only.** Every lens and every citation must trace to a real, fetched source. No invented studies, numbers, or URLs. If a figure can't be verified, demote or cut it; never paper over it.
- **The panel is author-built.** Always disclose this in the report. Agreement across lenses is a strong hypothesis, not independent proof. Do not present convergence as consensus of the field.
- **Verification is mandatory.** A report delivered without Phase 4 is not a Storm Research report. The verification banner must be truthful.
- **Reliability = evidence quality, not confidence.** Score on the source hierarchy: peer-reviewed causal > official policy/financial data > single commissioned survey > analogy > preprint.
- **Target the reader, not a default person.** The actionable insight and claim safety guide speak to the role identified in Phase 0. Keep them generic if no role is given.
- **Cost.** This spawns ~9-11 agents per run. That is expected. Do not fan out wider than five lenses or one verifier per citation cluster.
- **Design.** Modern editorial: dark plum hero, Space Grotesk display, Inter body, JetBrains Mono labels, violet (`#6d28d9`) + teal (`#0d9488`) accent. Keep the template CSS verbatim, including the `@page` / `@media print` block that makes the PDF paginate cleanly. Do not swap in a different visual style.
- **PDF is the only deliverable.** Always render and hand over the PDF. The HTML is a temporary render source written to the OS temp dir and discarded — never ship it or leave it in `storm-reports/`. Render only after Phase 4 corrections land so the PDF is the verified version.
- **Attribution.** The multi-perspective method is **inspired by** Stanford's STORM (Shao et al., NAACL 2024) — keep that credit in the report footer and do not strip it. Do not claim STORM's published benchmark numbers (e.g. "25% more organized") for *this* skill: those measured Stanford's own system in a human eval, not this five-lens/PDF adaptation. Credit the idea; don't borrow the scores.
