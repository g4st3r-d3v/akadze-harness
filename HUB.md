---
status: active
updated: 2026-10-05
tags:
  - akadze
  - hub
---

# akadze

Очередь задач на PostgreSQL в стиле Celery. В роли брокера Postgres: `SKIP LOCKED` и lease.

Код: https://github.com/g4st3r-d3v/akadze  
Харнесс (этот репозиторий): https://github.com/g4st3r-d3v/akadze-harness — единственный SoT по дизайну.

## Заметки

- [[Почему akadze]]
- [[Скетч продукта]]
- [[Roadmap wave 0]]

## Исследование (2026-10-05)

Перед кодом: что берём у Celery, что уже есть на рынке, риски Postgres как брокера и как ml-core ляжет на akadze.

- [[02 — Celery — что берём и что обходим]]
- [[03 — Ландшафт Postgres-очередей 2026]]
- [[04 — Postgres как брокер — риски и правила]]
- [[05 — ml-core как первый потребитель]]
- [[06 — Дизайн akadze v0]]

## Решения (2026-10-05)

1. Строим akadze. Первый потребитель: ml-core. План Б: Oban-py.
2. Слой базы в v0: SQLAlchemy 2 async Core.
3. В v0 только задачи: постановка, воркер, retry, snooze, отмена, periodic, обслуживание очереди.
4. Без HTTP/API-слоя. akadze: Python-вызовы, CLI, схема Postgres. HTTP/REST/web UI делает другая библиотека или приложение.
