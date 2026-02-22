# Отчёт по отладке приложения

ФИО: Баков Андрей Игоревич

1. Шаг 1: Анализ и запуск.

- Что сделал: Запустил `docker compose up --build -d`, посмотрел логи `docker compose logs`
- Проблема: Приложение падало на старте из-за некорректной конфигурации БД.

![Скриншот](./imgs/step_1_logs.png)

- Решение: Исправил настройки DATABASE_URL в app/core/config.py.

Было:

![Скриншот](./imgs/step_1_problem.png)

Стало:

![Скриншот](./imgs/step_1_patch.png)

Итог - приложение запускается

![Скриншот](./imgs/step_1_app_started.png)

2. Шаг 2: Повторный запуск (Ошибка парсинга).

- Что сделал: Запустил `docker compose up --build -d`, посмотрел логи `docker compose logs`
- Проблема: Парсинг падал из-за `city = null`.

![Скриншот](./imgs/step_2_logs.png)

- Причина:
  - обращение `item.city.name.strip()` без проверки `item.city` на `None`.
- Решение: Исправил код парсинга в app/services/parser.py.

Было:

![Скриншот](./imgs/step_2_problem.png)

Стало:

![Скриншот](./imgs/step_2_patch.png)

Итог - парсинг работает корректно

![Скриншот](./imgs/step_2_parser_work.png)

3. Шаг 3: Исправление бага (расписание парсинга).

- Описание проблемы: Фоновый парсинг запускался в секундах, а не в минутах.
- Файл: app/services/scheduler.py
- Причина: Неправильная единица времени в APScheduler.

Было:

![Скриншот](./imgs/step_3_problem.png)

Стало:

![Скриншот](./imgs/step_3_patch.png)

Итог - scheduler парсинга работает корректно

![Скриншот](./imgs/step_3_scheduler_work.png)

4. Шаг 4: Исправление бага (не закрывался HTTP-клиент).

- Описание проблемы: httpx.AsyncClient не закрывался, из-за чего в памяти приложения накапливался мусор.
- Файл: app/services/parser.py
- Причина: Отсутствовал механизм закрытия клиента.

Было:

![Скриншот](./imgs/step_4_problem.png)

Стало:

![Скриншот](./imgs/step_4_patch.png)


Итог - httpx.AsyncClient корректно закрывается

5. Шаг 5: Исправление бага (неверный статус при дубликате POST).

- Описание проблемы: При дубликате external_id API возвращал 200, а не конфликт.
- Файл: app/api/v1/vacancies.py

![Скриншот](./imgs/step_5_new_vac.png)
![Скриншот](./imgs/step_5_same_vac.png)

Возвращается статус 200, хотя логичнее возвращать ошибку 409 Conflict

Было:

![Скриншот](./imgs/step_5_problem.png)

Стало:

![Скриншот](./imgs/step_5_patch.png)

Итог - возвращается корректный статус код


![Скриншот](./imgs/step_5_return_409.png)

6. Шаг 6: Исправление бага (логика upsert по external_id).

- Описание проблемы: external_id=0 обрабатывался некорректно, тип existing_ids задан неверно.
- Файл: app/crud/vacancy.py
- Причина: Ошибка условий для nullable/integer-поля и неверный тип коллекции.

Было:

![Скриншот](./imgs/step_6_problem.png)

Стало:

![Скриншот](./imgs/step_6_patch.png)

Итог - поведение upsert стало корректным и предсказуемым для всех валидных external_id

7. Шаг 7: Исправление бага (конфликт при PUT).

- Описание проблемы: При обновлении вакансии на занятый external_id прилетал 500.
- Файл: app/api/v1/vacancies.py
- Причина: Необработанный unique conflict.

Сценарий ошибки:

Создадим 2 вакансии A и B
```json
{
    "title":"Bug7 A",
    "timetable_mode_name":"Гибкий",
    "tag_name":"backend",
    "city_name":"Москва",
    "published_at":"2026-02-22T20:00:00Z",
    "is_remote_available":true,
    "is_hot":false,
    "external_id":900001
}
```

![Скриншот](./imgs/step_7_A.png)


```json
{
    "title":"Bug7 B",
    "timetable_mode_name":"Фиксированный",
    "tag_name":"backend",
    "city_name":"СПб",
    "published_at":"2026-02-22T20:01:00Z",
    "is_remote_available":false,
    "is_hot":false,
    "external_id":900002
}
```

![Скриншот](./imgs/step_7_B.png)


Пробуем у B поставить external_id от A

![Скриншот](./imgs/step_7_put_500.png)

Было:

![Скриншот](./imgs/step_7_problem.png)

Стало:

![Скриншот](./imgs/step_7_patch.png)

Итог - корректно обрабатывается ситуация с дублирующимся external_id при обновлении

![Скриншот](./imgs/step_7_return_409.png)

8. Шаг 8: Исправление бага (`/api/v1/parse/` всегда 200).

- Описание проблемы: даже при внутренней ошибке парсинга всегда возвращается 200
- Файл: app/services/parser.py, app/api/v1/parse.py
- Причина: Ошибка бизнес-логики обработки исключений.

![Скриншот](./imgs/step_8_parser_error.png)

Фикс:

![Скриншот](./imgs/step_8_patch_1.png)
![Скриншот](./imgs/step_8_patch_2.png)


Итог - код возврата корректно отражает реальное состояние операции.

![Скриншот](./imgs/step_8_return_503.png)
