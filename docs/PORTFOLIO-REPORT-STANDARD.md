# Portfolio Investigation Report Standard

This shared structure applies to every repository and case study. It is a target for modernization, not a claim that older reports already comply.

## Case-study structure
1. **Header and status:** descriptive title, tools, scope, verified technique mappings.
2. **30-second executive summary:** question, method, finding, outcome.
3. **Environment and authorization:** hosts, data sources, controlled-simulation disclaimer.
4. **Timeline and investigation decisions:** what was observed, what was tested, and why.
5. **Evidence:** original screenshots embedded inline with captions and verified paths; never invent evidence.
6. **Technical walkthrough:** relevant commands, why each command was chosen, observed output, interpretation.
7. **Detection logic:** rule/script/query, test procedure, false positives and coverage limits.
8. **Response and validation:** only actions actually performed; unresolved questions.
9. **Five-minute refresher:** concise summary, command cheat sheet, memory cue and three self-test questions.
10. **Interview defense:** expandable Q&A on reasoning, alternatives, and limitations.
11. **Reproducibility:** links to original code, sanitized logs, evidence, and safe replication notes.

## Quality review
- Verify all screenshot links render and match the claim.
- Separate observed evidence from hypothesis; label simulations.
- Explain technical choices and what tools cannot prove.
- Keep readable GitHub Markdown with restrained visual badges.
- Preserve original investigation details and avoid fabricating missing source files.
- Verify all links, protect secrets, and retain limitations.

**Example:** [IR-013 systemd persistence investigation](https://github.com/kmon28/Kali-Linux-Security-Assessment-Lab/blob/main/reports/IR-013-Linux-Systemd-Persistence-Investigation.md).
