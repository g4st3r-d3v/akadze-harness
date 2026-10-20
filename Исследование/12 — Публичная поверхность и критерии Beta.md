---
status: active
updated: 2026-10-20
tags:
  - akadze
  - api
  - release
---

# Публичная поверхность и критерии Beta

Explanation для W5-2 из [[11 — Пул задач после 0.4.0]]. Здесь — что считается обещанием пакета, как оно меняется до 1.0 и когда пакет выходит из альфы. Политика версий — решение 12 в HUB.

## Что публично

Публичное — то, на что пользователь опирается без чтения исходников. Ломающее изменение этого списка помечается `!` в коммите и строкой в release notes; до 1.0 — только в миноре.

**Python.**
- Имена из `akadze.__all__` и их сигнатуры: `Akadze`, `Worker`, `current`, исключения (`AkadzeError`, `EnqueueError`, `DuplicateJob`, `Retry`, `Snooze`, `Fail`, `Cancel`), сводки (`JobSummary`, `LimitSummary`, `WorkerSummary`, `QueueSnapshot`, `StateCount`), запросы и команды (`list_jobs`, `list_limits`, `list_workers`, `queue_snapshot`, `requeue`, `request_cancel`, `pause_queue`, `resume_queue`, `set_limit`, `clear_limit`, `migrate`, `applied_versions`), тип `BackoffPolicy`.
- Методы `Akadze`: `task`, `register`, `periodic`, `include`, `add_schedule`, `add_schedules`, `aclose`, атрибуты `hooks` и `engine`.
- `Task.using(...)` и его опции, `BoundTask.enqueue` / `enqueue_many`.
- `Worker(...)` и его опции; `register`, `claim_available`, `step`, `serve`, `shutdown`.
- `current().complete_tx()`.
- Модули вне корня, которые документированы: `akadze.testing` (`drain`, `assert_enqueued`), `akadze.metrics.render_metrics`, `akadze.periodic.ScheduleSpec`.
- Поля `Job`, которые получают хуки (`on_enqueue`, `before_run`, `after_run`, `on_transition`), и сигнатуры хуков.

**CLI.** Подкоманды `worker`, `migrate`, `jobs list|requeue|cancel`, `queues pause|resume`, `limits list|clear`, `workers list`, `metrics [render]`, их флаги, формат строки вывода `key=value` и коды выхода (0 / 1 ошибка базы / 2 использование).

**Схема.** Таблицы `akadze.jobs`, `akadze.limits`, `akadze.workers`, `akadze.queue_pauses` в той части, которую читают ops-запросы и документация (состояния задачи, `limit_key`, `meta`, `run_at`, `attempt`, `max_attempts`). Миграции только добавляются, старые файлы не меняются.

**Метрики.** Имена и метки серий `akadze_jobs`, `akadze_queue_lag_seconds`, `akadze_worker_busy`, `akadze_queue_paused`.

## Что внутреннее

Модули и имена с `_`, SQL claim/finish/rescue, порядок CTE, `akadze.periodic_runs` и `akadze.limit_starts`, формат `errors` сверх `error`/`message`, текст сообщений исключений. Их можно менять в патче.

## Политика изменений до 1.0

- Минор — волна или ломающее изменение публичного. Патч — исправления и небольшие аддитивные изменения, нужные потребителю раньше волны.
- Смена поведения по умолчанию, на которое опираются (порядок claim, кривая backoff), — минор и строка в release notes, даже если сигнатуры те же.
- Удаление публичного имени — сначала `DeprecationWarning` в одном миноре, удаление в следующем.

## Критерии Beta

Beta — обещание, что поверхность выше меняется только по политике deprecation. Выходим, когда всё ниже выполнено:

1. Этот список записан, а тест в пакете сверяет `akadze.__all__` с ним (поломка — красный CI, а не сюрприз в релизе).
2. Минимум один потребитель в проде на текущем миноре без обходных путей и форков.
3. Нет открытых багов класса «задача потеряна» или «задача выполнена дважды без падения воркера».
4. How-to миграции с Celery соответствует текущему API (W5-3).
5. Claim на очереди в 100 тыс. готовых задач укладывается в десятки миллисекунд в обоих режимах, чтобы обещание «Postgres как брокер» держалось на реальном хвосте.

Пункты 1, 4 закрыты Wave 5. Пункт 5 закрыт для порядка по умолчанию (W5-9); для `fair` — следующая волна. Пункты 2–3 проверяет владелец перед объявлением Beta; номер версии Beta — решение владельца.
