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

