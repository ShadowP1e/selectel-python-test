# Отчёт по отладке приложения

ФИО: Баков Андрей Игоревич

1. Шаг 1: Анализ и запуск.

- Что сделал: Запустил `docker compose up --build -d`, посмотрел логи `docker compose logs`
- Проблема: Приложение падало на старте из-за некорректной конфигурации БД.

![alt text](./imgs/step_1_logs.png)

- Решение: Исправил настройки DATABASE_URL в app/core/config.py.

Было:

![alt text](./imgs/step_1_problem.png)

Стало:

![alt text](./imgs/step_1_patch.png)

Итог - приложение запускается

![alt text](./imgs/step_1_app_started.png)

2. Шаг 2: Повторный запуск (Ошибка парсинга).

- Что сделал: Запустил `docker compose up --build -d`, посмотрел логи `docker compose logs`
- Проблема: Парсинг падал из-за `city = null`.

![alt text](./imgs/step_2_logs.png)

- Причина:
  - обращение `item.city.name.strip()` без проверки `item.city` на `None`.
- Решение: Исправил код парсинга в app/services/parser.py.

Было:

![alt text](./imgs/step_2_problem.png)

Стало:

![alt text](./imgs/step_2_patch.png)

Итог - парсинг работает корректно

![alt text](./imgs/step_2_parser_work.png)

3. Шаг 3: Исправление бага (расписание парсинга).

- Описание проблемы: Фоновый парсинг запускался в секундах, а не в минутах.
- Файл: app/services/scheduler.py
- Причина: Неправильная единица времени в APScheduler.

Было:

![alt text](./imgs/step_3_problem.png)

Стало:

![alt text](./imgs/step_3_patch.png)

Итог - scheduler парсинга работает корректно

![alt text](./imgs/step_3_scheduler_work.png)

4. Шаг 4: Исправление бага (не закрывался HTTP-клиент).

- Описание проблемы: httpx.AsyncClient не закрывался, из-за чего в памяти приложения накапливался мусор.
- Файл: app/services/parser.py
- Причина: Отсутствовал механизм закрытия клиента.

Было:

![alt text](./imgs/step_4_problem.png)

Стало:

![alt text](./imgs/step_4_patch.png)


Итог - httpx.AsyncClient корректно закрывается

5. Шаг 5: Исправление бага (неверный статус при дубликате POST).

- Описание проблемы: При дубликате external_id API возвращал 200, а не конфликт.
- Файл: app/api/v1/vacancies.py

![alt text](./imgs/step_5_new_vac.png)
![alt text](./imgs/step_5_same_vac.png)

Возвращается статус 200, хотя логичнее возвращать ошибку 409 conflict

Было:

![alt text](./imgs/step_5_problem.png)

Стало:

![alt text](./imgs/step_5_patch.png)

Итог - возвращается корректный статус код


![alt text](./imgs/step_5_return_409.png)
