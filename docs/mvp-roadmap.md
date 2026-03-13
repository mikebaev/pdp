# MVP Roadmap

## Phase 1 — Data ingestion and normalization
- [ ] CV/LinkedIn/manual input adapters.
- [ ] Unified schema storage (`role`, `period`, `context`, `stack`, `results`).
- [ ] Vacancy parser with must/nice separation.

Deliverable: normalized candidate and vacancy objects.

## Phase 2 — Decomposition and gap analysis
- [ ] Skill inventory extraction with evidence linking.
- [ ] Vacancy keyword map (`synonyms`, `spellings`, `abbreviations`).
- [ ] Gap matrix (`covered/missed/unknown`) with explanation.

Deliverable: transparent match model between profile and job requirements.

## Phase 3 — ATS augmentation and truth control
- [ ] Derived baseline skills generator (for implicit senior competencies).
- [ ] `strict_truth` and `needs_confirm` workflows.
- [ ] Validation gate before export.

Deliverable: safe ATS keyword layer without factual distortion.

## Phase 4 — Resume composition
- [ ] Length presets (`short`, `standard`, `extended`).
- [ ] Bullet generator (`achievement -> metric -> tool`) with fact lock.
- [ ] Split output: recruiter-facing narrative + ATS Core Skills block.

Deliverable: job-tailored resume variants.

## Phase 5 — Quality checks and export
- [ ] ATS match score + breakdown.
- [ ] Actionable suggestions for iteration.
- [ ] Export: ATS-friendly text/markdown, then DOCX/PDF in next increment.

Deliverable: ready-to-send artifacts with explainable score.
