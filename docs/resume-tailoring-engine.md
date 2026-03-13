# Resume Tailoring Engine

## 1. Цель продукта
Создать инструмент «Декомпозиция → Дополнение → Композиция», который:
- преобразует неструктурированный опыт кандидата в проверяемую базу фактов;
- сопоставляет эту базу с вакансией на уровне требований и ключевых слов;
- собирает две версии результата: человеко-читаемое резюме и ATS-слой ключевых навыков;
- исключает добавление неподтвержденных фактов.

## 2. Продуктовые принципы
1. **Truth-first** — любое утверждение должно иметь источник в профиле/опыте.
2. **Dual-output** — отдельно оптимизируем читабельность для рекрутера и машиночитаемость для ATS.
3. **Evidence-linked** — каждый навык и буллет привязан к доказательству.
4. **Compression-aware** — при ограничении длины сохраняются must-требования и strongest evidence.
5. **Iterative** — каждая итерация улучшает match score без фальсификаций.

## 3. Границы MVP

### Входит в MVP
- Импорт опыта (CV/LinkedIn/manual text) в унифицированную структуру.
- Импорт текста вакансии и выделение must/nice требований.
- Извлечение навыков из опыта с привязкой к доказательствам.
- Keyword map вакансии с синонимами.
- Gap analysis с состояниями covered/missed/unknown.
- Генерация резюме (краткая и расширенная версии).
- ATS score с объяснимым breakdown.
- Экспорт в ATS-friendly plain text + markdown.

### Не входит в MVP
- Полноценный визуальный editor шаблонов DOCX/PDF.
- Автоматическое парсинг по URL джоббордов с anti-bot обходами.
- ML-обучение на outcome feedback (это v2).

## 4. Доменная модель

### CandidateProfile
- `id`
- `headline`
- `seniority`
- `roles[]`
- `projects[]`
- `achievements[]`
- `raw_sources[]`

### ExperienceEvidence
- `id`
- `source_type` (`cv`, `linkedin`, `manual`)
- `snippet`
- `role`
- `period`
- `context`
- `stack[]`
- `result_metric`
- `confidence` (0..1)

### SkillInventoryItem
- `skill_canonical`
- `variants[]`
- `evidence_ids[]`
- `strength` (`strong`, `medium`, `weak`)
- `derived` (bool)
- `derivation_rule` (nullable)

### VacancyRequirement
- `id`
- `type` (`must`, `nice`)
- `text`
- `keywords[]`
- `synonyms[]`
- `domain`
- `seniority_signal`

### GapMatrixItem
- `requirement_id`
- `status` (`covered`, `missed`, `unknown`)
- `matched_skills[]`
- `evidence_ids[]`
- `clarification_question` (nullable)

### ResumeArtifact
- `target_role`
- `length_preset` (`short`, `standard`, `extended`)
- `summary`
- `experience_bullets[]`
- `core_skills_keywords[]`
- `truth_mode` (`strict_truth`, `needs_confirm`)

## 5. Workflow (Декомпозиция → Дополнение → Композиция)

### Этап A. Нормализация
1. Принять входные данные (файл/текст/вставка).
2. Разметить сущности: роль, проект, стек, результат.
3. Сохранить в `CandidateProfile` + `ExperienceEvidence`.

### Этап B. Декомпозиция
1. Извлечь `SkillInventoryItem` из evidence.
2. Построить `KeywordMap` вакансии (ключ + варианты написания + синонимы).
3. Связать требования вакансии с навыками кандидата.

### Этап C. Дополнение
1. Сформировать gap matrix.
2. Для `unknown` предложить уточняющие вопросы.
3. Для implicit-базовых навыков (например, Git, CI/CD у CTO) добавить `derived=true` и `derivation_rule`.

### Этап D. Композиция
1. Сгенерировать summary и bullets по шаблону «достижение → метрика → инструмент».
2. Внедрить релевантные ключевые слова в естественных формулировках.
3. Собрать ATS-слой в отдельный блок `Core Skills / Keywords`.
4. Сжать контент по preset длины без потери must coverage.

### Этап E. Проверка
1. Рассчитать score по must/nice.
2. Показать explanation (что покрыто/что отсутствует/где улучшить).
3. В режиме `strict_truth` удалить неподтвержденные элементы.

## 6. Логика ATS score

Рекомендуемая формула:

`total_score = 0.7 * must_coverage + 0.2 * nice_coverage + 0.1 * keyword_density_quality`

Где:
- `must_coverage`: доля must-требований со статусом `covered`;
- `nice_coverage`: доля nice-требований со статусом `covered`;
- `keyword_density_quality`: штрафует за keyword stuffing и награждает за естественное распределение.

Дополнительно:
- Отдельный `truth_penalty`, если элемент без evidence прошел в draft.
- `confidence_overlay` для подсветки слабых совпадений.

## 7. Режимы правды

### strict_truth
- В output включаются только элементы с `evidence_ids.length > 0`.
- `derived` допускается, только если есть `derivation_rule` + базовое доказательство.

### needs_confirm
- Разрешает пометить пункт как `needs_confirmation`.
- Перед экспортом пользователь должен подтвердить/отклонить.

## 8. Acceptance coverage (traceability)

| US | Реализация в движке |
|---|---|
| US1 | Parser + Normalizer + CandidateProfile schema |
| US2 | Vacancy parser + must/nice classifier + entity extraction |
| US3 | Skill extraction + evidence linkage + strength |
| US4 | Keyword map + synonyms/variants |
| US5 | Gap matrix + evidence-backed status |
| US6 | Core Skills layer + derived tags |
| US7 | Truth modes + validation gate |
| US8 | Length presets + compression policy |
| US9 | Bullet generator with fact-lock |
| US10 | ATS score + explainability report |
| US11 | ATS-friendly exports |
| US12 | Version history + outcome analytics (v2) |

## 9. Нефункциональные требования
- **Explainability:** каждое авто-решение сопровождается причиной.
- **Auditability:** можно восстановить, из каких данных собран финальный буллет.
- **Latency:** цель MVP — генерация за < 20 секунд на обычный профиль.
- **Privacy:** персональные данные и резюме шифруются at rest.

## 10. Метрики успеха
1. Рост ATS-pass-rate на пилотной группе.
2. Снижение времени адаптации резюме под вакансию.
3. Доля generated bullets, принятых пользователем без ручного редактирования.
4. Количество false-positive навыков (цель — близко к 0 в strict mode).
