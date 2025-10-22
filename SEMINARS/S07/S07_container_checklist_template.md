# S07 - Containerization Checklist (template)

> Цель: аккуратно «упаковать» one-liner из S06 в контейнер (Docker/Compose), собрать проверяемые артефакты и обновить DV.

## Как пользоваться

1. Пройдитесь по разделам, ставьте галочки и добавляйте **точные ссылки** на артефакты из `EVIDENCE/S07/`.
2. По завершении перенесите краткую сводку в `GRADING/DV.md` (DV2/DV4/DV5).
3. **Не коммитьте секреты.** Для примеров используйте `.env.example`.

---

## Паспорт выполнения (заполнить)

* Команда/репозиторий: https://github.com/phamtienmanh2001/secdev-s06-s08
* Тег образа: **secdev-seed:pin**
* Вариант запуска: ☐ Docker  x Compose
* One-liner из S06 (для справки): **python -m venv .venv; .\.venv\Scripts\Activate.ps1; pip install -r requirements.txt; python scripts/init_db.py; pytest -q --junitxml=EVIDENCE/S06/test-report.xml**
* Ссылки на артефакты S07: см. «Индекс пруфов».

---

## A. Образ (build)

* [x] **python:3.11-slim**, зафиксирован тег/дистрибутив.
  *Комментарий:* Offical
* [x] **`.dockerignore`** присутствует и исключает мусор (`__pycache__/`, `.venv/`, `tests/`, `EVIDENCE/`, `.git/`, `.env`).
* [x] **WORKDIR, COPY, RUN**: зависимости ставятся из `requirements.txt` с `--no-cache-dir`; копируются только нужные файлы (app/, scripts/, requirements.txt).
* [x] **Сборка воспроизводима**: командой

  ```bash
  docker build -t secdev-seed:pin . | tee EVIDENCE/S07/build.log
  ```

* [x] **Лог сборки сохранён** → `EVIDENCE/S07/build.log`.
* [x] **Размер образа зафиксирован** → `EVIDENCE/S07/image-size.txt`.
* [ ] (Опц.) **Multi-stage**/очистка слоёв - если применимо, описать.
  *Комментарий:* **ТУТ НУЖНА ТВОЯ ВСТАВКА**

**DV:** DV4 (артефакты); DV5 (описание сборки в README).

---

## B. Запуск (run)

* [x] Порт проброшен (`-p 8000:8000`) или описан в Compose.
* [x] Контейнер поднимается командой:

  ```bash
  docker compose up --build | tee EVIDENCE/S07/compose-up.log
  ```

* [x] **Лог запуска сохранён** → `EVIDENCE/S07/compose-up.log`.
* [x] **Доступность проверена** (`GET /` → 200) → `EVIDENCE/S07/http_root_code.txt`.
* [x] **inspect контейнера сохранён** → `EVIDENCE/S07/inspect_web.json`.
* [x] (Если есть healthcheck) **статус сохранён** → `EVIDENCE/S07/health.json` (или скрин).

**DV:** DV4 (артефакты); DV5 (README: раздел «Запуск в контейнере»).

---

## C. Конфиги/секреты (гигиена)

* [x] **`.env` не в репозитории**, только `.env.example` (образец значений).
* [x] Значения конфигов читаются из **переменных окружения**; секреты не попадают в образ/логи.
* [x] В Compose/CI используются секреты/vars платформы (не хранить значения в YAML).
* [x] В `README` коротко описано: **какие переменные** нужны для запуска.

**DV:** DV2 (гигиена секретов); DV5 (README).

---

## D. Рантайм-безопасность (лайт-харденинг)

Выберите **минимум 1-2** пункта (для ★★ лучше 2):

* [x] **Non-root user** в контейнере (пример для Debian slim):

  ```Dockerfile
  # пример, адаптируйте под свою базу
  RUN useradd -m -u 10001 appuser
  USER appuser
  ```

  Доказательство: `docker exec seed id -u` → `EVIDENCE/S07/non-root.txt`.
* [x] **Healthcheck** (в Dockerfile или Compose). Пруф: `health.json`.
* [ ] **Read-only FS** и явный writable-каталог, если нужно (опц.).
* [ ] **Минимизация поверхности**: укажите, что именно исключили/не устанавливали.
* [ ] (Опц.) Линт Dockerfile (`hadolint`) → `EVIDENCE/S07/hadolint.txt`.

**DV:** DV4 (артефакты/логи); комментарий в DV5.

---

## E. Compose (если используется)

* [x] Сервис `web` с `build: .`/`image:`, `ports: "8000:8000"`.
* [x] `environment:` - только неспецифичные/несекретные значения; реальные - через секреты/vars.
* [x] `healthcheck:` задан (test/interval/timeout/retries).
* [ ] (Опц.) `volumes:` для `app.db`/кэшей - если нужна локальная сохранность.
* [x] `restart:` политика задана.
* [x] Лог `compose up` сохранён → `EVIDENCE/S07/compose-up.log`.

**DV:** DV4 (артефакты); DV5 (README: пример `docker compose up --build`).

---

## Индекс пруфов (указывать точные пути)

* [x] `EVIDENCE/S07/build.log`
* [x] `EVIDENCE/S07/image-size.txt`
* [x] `EVIDENCE/S07/run.log` **или** `EVIDENCE/S07/compose-up.log`
* [x] `EVIDENCE/S07/http_root_code.txt`
* [x] `EVIDENCE/S07/inspect_web.json`
* [x] `EVIDENCE/S07/health.json` *(если есть healthcheck)*
* [x] `EVIDENCE/S07/non-root.txt` *(если делали non-root)*
* [ ] `EVIDENCE/S07/hadolint.txt` *(если запускали linт)*

---

## Сводка для переноса в `GRADING/DV.md`

* **DV2 (гигиена секретов):** `.env` не коммитим; используем `.env.example`; секреты передаются окружением/секрет-хранилищем.
* **DV4 (артефакты/логи):** перечислите файлы из `EVIDENCE/S07/` (см. индекс выше).
* **DV5 (инструкции):** в `README` добавлен раздел «Запуск в контейнере» с командами:

  ```bash
  docker build -t secdev-seed:latest .
  docker run --rm -p 8000:8000 --name seed secdev-seed:latest
  # или:
  docker compose up --build
  ```

> **Примечание:** **DV1** (one-liner локальной сборки/тестов) остался из S06 и в Docker **не переносится**.

---

## Definition of Done (минимум/проектный)

**Минимум (★1):**

* [x] Образ собирается; контейнер отвечает `200` на `/`.
* [x] В `EVIDENCE/S07/` есть build/run/inspect (+ health при наличии).
* [x] `GRADING/DV.md` и `README` обновлены (DV2/DV4/DV5).

**Проектный (★★2):**

* [x] Применены ≥2 пункта харденинга (например, non-root + healthcheck).
* [x] Обоснован размер образа (коммент в DV) и/или снижено число слоёв.
* [x] (Опц.) Пройден линт Dockerfile; приложены отчёты.

---

**Подсказка:** если порт 8000 занят - временно используйте `-p 18000:8000` и зафиксируйте это в `README` и `EVIDENCE/S07/http_root_code.txt`.
