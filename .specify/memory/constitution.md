<!--
Sync Impact Report
- Version change: template -> 1.0.0
- Modified principles:
  - Placeholder principles -> I. Backend-First .NET
  - Placeholder principles -> II. React Only for a Required UI
  - Placeholder principles -> III. Reference-Driven OCR (NON-NEGOTIABLE)
  - Placeholder principles -> IV. Human Review and Truthful Results
  - Placeholder principles -> V. Data Minimization and Verifiable Quality
- Added sections:
  - Technology and OCR Constraints
  - Development Workflow and Quality Gates
- Removed sections: none
- Templates:
  - ✅ .specify/templates/plan-template.md
  - ✅ .specify/templates/spec-template.md
  - ✅ .specify/templates/tasks-template.md
  - ✅ .agents/skills/speckit-tasks/SKILL.md
  - ✅ AGENTS.md
- Follow-up TODOs: none
-->
# VibeCehDemo1_1 Constitution

## Core Principles

### I. Backend-First .NET
Серверная часть и бизнес-логика MUST реализовываться на C# и .NET. Доменная
логика MUST быть отделена от API, инфраструктуры, OCR-адаптеров и хранения
данных так, чтобы критические правила можно было тестировать без запуска UI или
внешних процессов. Версия .NET, структура solution и ключевые зависимости MUST
быть явно зафиксированы в плане каждой feature. Новая технология на backend
допускается только при документированной невозможности решить задачу средствами
.NET и после review.

### II. React Only for a Required UI
Frontend не является обязательной частью решения. Если пользовательский
интерфейс входит в утверждённый scope, он MUST быть реализован на React и
взаимодействовать с backend через явно описанные контракты. Бизнес-правила,
валидация подтверждения и OCR processing MUST оставаться на backend; React не
может быть единственным местом их реализации. Добавление frontend без
пользовательского сценария и критериев приёмки запрещено.

### III. Reference-Driven OCR (NON-NEGOTIABLE)
Для passport OCR файл `03-passport-ocr-reference-constraints.md` является
источником истины. Минимальная реализация MUST использовать локальный Tesseract
с `rus.traineddata`, PDF rasterization с baseline 300 DPI, обязательный image
preprocessing и zonal OCR. Нельзя молча заменять OCR engine, подключать cloud
OCR, обходить preprocessing или менять baseline zones. Любое изменение
reference MUST быть отдельным решением с причиной, влиянием на тесты,
повторным запуском OCR tests и review до реализации.

### IV. Human Review and Truthful Results
OCR workflow MUST поддерживать ручную проверку и исправление результата перед
подтверждением. Только `passportSeries` и `passportNumber` обязательны для
confirm; неоднозначные необязательные поля MUST оставаться пустыми, а не
угадываться. Частичный OCR является допустимым результатом и MUST отличаться от
технического сбоя pipeline и ошибки валидации confirm. Автоматическое
подтверждение паспорта без участия пользователя запрещено в минимальном scope.

### V. Data Minimization and Verifiable Quality
Система MUST сохранять только подтверждённые паспортные поля, минимальную
metadata попытки, статус и безопасное публичное сообщение об ошибке. Исходные
PDF/PNG, raw OCR text, crops, debug images и preprocessing artifacts не должны
сохраняться постоянно; временные файлы MUST очищаться после обработки.
Логи MUST исключать паспортные реквизиты и содержимое документов. Контракты,
нормализация, state transitions, очистка временных данных и OCR pipeline MUST
иметь автоматизированные тесты; изменения baseline zones MUST проверяться на
sample documents.

## Technology and OCR Constraints

- Backend: C# и .NET; конкретная поддерживаемая версия фиксируется в plan.md.
- Frontend при наличии утверждённого UI: React; версия и toolchain фиксируются
  в plan.md.
- Вход минимального OCR workflow: ровно один PDF или PNG на попытку.
- PDF rasterization baseline: 300 DPI.
- OCR: локальный Tesseract, язык `rus`, файл `rus.traineddata`.
- Pipeline: normalization, preprocessing, zonal OCR, extraction, manual review,
  confirm, persistence.
- Поля, зоны, координаты, форматы нормализации, failure behavior, storage policy
  и out-of-scope определяются в `03-passport-ocr-reference-constraints.md`.
- Authentication, batch processing, cloud OCR, несколько типов документов и
  production-grade accuracy не входят в минимальный scope без отдельной spec.

## Development Workflow and Quality Gates

Каждая feature MUST пройти последовательность spec -> clarify при необходимости
-> plan -> tasks -> implement. Spec MUST содержать independently testable user
stories, measurable acceptance criteria, явный scope и ссылки на применимые
reference-документы. Plan MUST фиксировать версии .NET/React, архитектурные
границы, внешние/native зависимости, обработку персональных данных и результаты
Constitution Check.

Для OCR-feature plan и tasks MUST обеспечивать трассируемость к
`03-passport-ocr-reference-constraints.md`. До реализации MUST быть определены
контракт результата, состояния processing attempt, очистка временных artifacts
и тестовые сценарии для PDF, PNG, partial OCR, technical failure и confirm
validation. Pull request или review MUST подтвердить прохождение тестов,
отсутствие утечки персональных данных и отсутствие необоснованных отклонений от
конституции. Любое отклонение MUST быть записано в Complexity Tracking с
причиной и отклонённой более простой альтернативой.

## Governance

Эта конституция имеет приоритет над локальными привычками и неявными
архитектурными решениями. Изменение принципов требует обновления этого файла,
Sync Impact Report, проверки зависимых Spec Kit шаблонов и review. Версия
изменяется по Semantic Versioning: MAJOR для несовместимого удаления или
переопределения принципов, MINOR для нового принципа или существенного
расширения обязательств, PATCH для уточнений без изменения требований.

Каждая spec и plan MUST проходить Constitution Check, а каждая реализация и
review MUST подтверждать соблюдение применимых gates. Технические детали OCR
изменяются сначала в `03-passport-ocr-reference-constraints.md`, затем
синхронизируются со spec, plan, tasks и тестами.

**Version**: 1.0.0 | **Ratified**: 2026-06-11 | **Last Amended**: 2026-06-11
