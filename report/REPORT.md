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
