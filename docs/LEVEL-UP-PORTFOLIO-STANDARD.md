# LEVEL UP — Cybersecurity Portfolio Operating Standard

**Status:** Active cross-repository editorial and engineering standard. **Quality target:** 95+/100 per technical repository and across the combined portfolio, **only after evidence-backed verification**. This is an internal rubric, not an external recruiter or ATS score.

## Mission

Show the work of a thoughtful SOC analyst: clear detection hypotheses, trustworthy evidence, defensible conclusions, and usable code. Write with experienced analyst-level precision **without claiming professional incident-response employment or real compromise from home-lab simulations**.

**Voice:** Direct, natural, specific, human. Explain technical terms plainly without diluting accuracy. Avoid marketing hype, AI-sounding repetition, exaggerated claims, boilerplate and generic stock imagery.

**Visual identity — SOC // FIELD NOTES:** Clean, modern, distinctive and restrained. Dark-theme-friendly Markdown; concise navigation; well-spaced tables; meaningful diagrams; a few consistent blue/teal accents; original clickable screenshots close to the relevant finding. GitHub Markdown must remain readable on mobile and in light mode. Avoid badge clutter and walls of text.

## Three audiences

1. **Recruiter (30 seconds):** What was investigated, tools, measurable result, demonstrated skill.
2. **SOC manager (3 minutes):** Hypothesis, triage, event sequence, context, verdict, response, limitations.
3. **Security engineer (10+ minutes):** Original evidence, exact commands, telemetry, source, tests, detection tradeoffs and reproducibility.

## Report architecture (adapt to the investigation)

1. Title, status, scope and a concise **plain-English overview**.
2. Executive summary: question, finding, confidence, verified outcome.
3. Lab topology and data sources (event IDs, log channels, capture vantage points).
4. Investigation chronology and reasoning: what prompted each next step.
5. Original, inline, clickable screenshot evidence with a caption explaining **what it shows, why it matters, and what it does not prove**.
6. Detection logic, scripts and commands with inputs, prerequisites and expected outputs.
7. False positives, blind spots, alternative explanations and ATT&CK mappings **only where substantiated**.
8. Containment / remediation / recovery **only if actually performed**, plus verification.
9. Reproduction instructions, source provenance, limitations, and analyst takeaways.
10. Optional concise interview questions / teaching notes, separate from the investigative record.

## Source and evidence integrity

- Never invent log output, screenshot details, test results, timestamps, incident outcomes or attacker intent.
- Clearly distinguish **observed fact**, **analyst inference**, and **untested hypothesis**.
- Controlled simulations are not real compromises; an alert match is not proof of malicious activity.
- Distinguish **original lab script**, **reconstructed script**, and **new portfolio reference implementation**. Uploading code does not prove it ran.
- Test new code against safe fixtures; document pass/fail honestly. Prefer free tools and standard-library approaches when practical.
- Do not publish passwords, tokens, sensitive challenge-response material, or unredacted personal data.
- Preserve historical work and numbering; improve in place rather than erasing provenance.
- When a screenshot is newly needed, specify its exact filename and save location.
- Splunk artifacts document past trial use; the expired trial is **not** an active service.

## Scoring rubric

| Criterion | Points |
| --- | ---: |
| Plain-English clarity | 15 |
| Technical accuracy and claim discipline | 20 |
| Original evidence and accessibility | 20 |
| Investigation / detection depth | 15 |
| Reproducibility and source availability | 15 |
| Response / outcome verification | 10 |
| Navigation and visual presentation | 5 |
| **Total** | **100** |

**95+ is a goal, not an automatic score.** Grade each repository separately, then grade cross-repository cohesion. An audit must check broken links, image rendering, code syntax/tests, factual claims, source provenance and technical completeness. Report unverified areas rather than silently granting points.

## Workflow for every future update

1. Inspect current GitHub content and relevant prior lab records before editing.
2. Preserve completed work; do not make the learner redo verified steps.
3. Prioritize substantive evidence, detection quality, working scripts and reproducibility over cosmetic edits.
4. Commit directly to the connected GitHub repositories where supported.
5. Re-fetch changed files to verify commits; never claim full test coverage without running tests.
6. Maintain a clear changelog, scorecard, open gaps and next action.
7. Explain each command at an approachable level: what, why, important arguments, expected output and common errors.
8. Add scenario questions so the portfolio owner can defend the work independently in a SOC interview.

## Portfolio map

- [Main profile](https://github.com/kmon28)
- [Windows SOC Detection](https://github.com/kmon28/Windows-SOC-Detection-Lab)
- [Network Traffic Analysis](https://github.com/kmon28/Network-Traffic-Analysis-Lab)
- [Kali / Linux Security Assessment](https://github.com/kmon28/Kali-Linux-Security-Assessment-Lab)

**Guiding principle:** Build it like a SOC analyst. Document it like a security engineer. Explain it like a great teacher. Present it like a professional.
