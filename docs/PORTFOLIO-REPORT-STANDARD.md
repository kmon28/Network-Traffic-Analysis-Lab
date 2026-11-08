# Technical Case-Study Presentation Standard

This is the common public-facing format for the three security lab repositories. It is a quality target; older reports are being updated incrementally.

1. **Project overview:** purpose, environment, threat model, tools, outcome, and skills demonstrated.
2. **Executive summary:** concise question, investigation, key findings, and disposition.
3. **Scope and lab topology:** authorization, assets, assumptions, and simulation status.
4. **Methodology and timeline:** analyst actions and the technical reason for each decision.
5. **Inline original evidence:** verified screenshots immediately beside interpretation, with meaningful captions.
6. **Detection and engineering:** commands, queries, source code, test results, false positives, and limitations.
7. **Response and validation:** only actions actually performed, including verification.
8. **Technical Summary & Key Takeaways:** short recap and command reference.
9. **Engineering Decisions & Trade-offs:** concise expandable rationale suitable for a technical reviewer.
10. **Reproducibility and scope limits:** working source/evidence links, honest uncertainty, no invented results.

## Audience separation

**Public reports** are professional security investigation documents. Avoid headings such as “Interview Refresher,” “Self-Test,” or “Interview Defense” in the main narrative. Keep useful technical review questions as engineering considerations, or place practice questions in a dedicated [study guide](TECHNICAL-STUDY-GUIDE.md).

**Personal study guides** may include flashcards, active recall, interview questions, and concise command explanations.

## Final review checklist

- Every linked screenshot exists and is legible; every inline image resolves.
- Observations, hypotheses, and proposed future work are clearly distinguished.
- MITRE mappings and security verdicts match the evidence.
- No secrets, credentials, or sensitive authentication material are exposed.
- Tools that have expired or are unavailable are described historically.
- Original code and raw artifacts are published only when available and safe.
- Formatting works on GitHub desktop and mobile.

**Reference implementation:** [IR-013 Linux systemd investigation](https://github.com/kmon28/Kali-Linux-Security-Assessment-Lab/blob/main/reports/IR-013-Linux-Systemd-Persistence-Investigation.md).
