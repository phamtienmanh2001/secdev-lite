# DV - Мини-проект «DevOps-конвейер»

> Этот файл - **индивидуальный**. Его проверяют по **rubric_DV.md** (5 критериев × {0/1/2} → 0-10).
> Подсказки помечены `TODO:` - удалите после заполнения.
> Все доказательства/скрины кладите в **EVIDENCE/** и ссылайтесь на конкретные файлы/якоря.

---

## 0) Мета

- **Проект (опционально BYO):** «учебный шаблон»
- **Версия (commit/date):** 8471e1db727311901b717f4fd50bd1ac3f98d7c4 / 2025-10-21
- **Кратко (1-2 предложения):** «учебный шаблон» с применением патчей S06-01 - SQLi (login), S06-02 - SQLi (search LIKE), S06-03 - XSS (экранирование вывода), S06-04 - Валидация ввода (строгость/длины/паттерны)

---

## 1) Воспроизводимость локальной сборки и тестов (DV1)

- **Одна команда для сборки/тестов:**

  ```bash
  python -m venv .venv; .\.venv\Scripts\Activate.ps1; pip install -r requirements.txt; python scripts/init_db.py; pytest -q --junitxml=EVIDENCE/S06/test-report.xml
  ```

  _Если без Makefile: укажите последовательность команд._


* ОС/шелл: Windows 11 / PowerShell 5.1+
* Версия Python: 3.13.1
- **Описание шагов (кратко):** 

1. Скачайте проект.
2. Убедитесь, что на вашем компьютере установлены Windows 11, Python 3.11 и PowerShell 5.1+.
3. Откройте PowerShell в папке скачанного проекта.
4. Скопируйте и запустите одну команду
  ```bash
  python -m venv .venv; .\.venv\Scripts\Activate.ps1; pip install -r requirements.txt; python scripts/init_db.py; pytest -q --junitxml=EVIDENCE/S06/test-report.xml
  ```

---

## 2) Контейнеризация (DV2)

- **Dockerfile:** https://github.com/phamtienmanh2001/secdev-s06-s08/blob/main/Dockerfile (минимальный образ с улучшением)
- **Сборка/запуск локально:**

  ```bash
  docker build -t secdev-seed:pin .
  docker run --rm -p 8000:8000 --name secdev-seed secdev-seed:pin
  ```

- **Порт:** 8000
- **Healthcheck:**

  ```bash
  docker inspect secdev-seed | jq '.[0].State.Health' > EVIDENCE/S07/health.json
  ```
- **HTTP root code:**
  ```bash
  curl.exe -sS http://127.0.0.1:8000/ -o NUL -w "%{http_code}\n" > EVIDENCE/S07/http_root_code.txt
  ```
- **Метаданные контейнера:**
  ```bash
  docker inspect secdev-seed > EVIDENCE/S07/inspect_web.json
  ```
- **Размер образа:**
  ```bash
  docker image ls secdev-seed:pin --format "{{.Repository}} {{.Tag}} {{.Size}}" > EVIDENCE/S07/image-size.txt
  ```
  _Укажите порт/команду/healthcheck, если есть._

- **(Опционально) docker-compose:** https://github.com/phamtienmanh2001/secdev-s06-s08/blob/main/docker-compose.yml - онли веб-сервис 

- **Что сделано:** включены в Dockerfile healthcheck, non-root user

---

## 3) CI: базовый pipeline и стабильный прогон (DV3)

- **Платформа CI:** GitHub Actions
- **Файл конфига CI:** `.github/workflows/ci.yml`
- **Стадии (минимум):** Checkout → Setup Python → Cache pip → Install deps → Init DB → Run tests with coverage → Upload artifacts
- **Фрагмент конфигурации (ключевые шаги):**

  ```yaml
  name: ci

  on:
    push:
      paths-ignore:
        - 'EVIDENCE/**'
        - 'README.md'
    pull_request:
      paths-ignore:
        - 'EVIDENCE/**'
        - 'README.md'

  concurrency:
    group: ci-${{ github.ref }}
    cancel-in-progress: true

  jobs:
    build-test:
      runs-on: ubuntu-latest
      strategy:
        fail-fast: false
        matrix:
          python-version: ['3.11', '3.12']
      timeout-minutes: 15
      env:
        PYTHONUNBUFFERED: "1"
      steps:
        - name: Checkout
          uses: actions/checkout@v4

        - name: Setup Python
          uses: actions/setup-python@v5
          with: 
            python-version: ${{ matrix.python-version }} 

        - name: Cache pip
          uses: actions/cache@v4
          with:
            path: ~/.cache/pip
            key: ${{ runner.os }}-pip-${{ hashFiles('**/requirements.txt') }}
            restore-keys: |
              ${{ runner.os }}-pip-

        - name: Install deps
          run: pip install -r requirements.txt

        - name: Init DB
          run: python scripts/init_db.py

        - name: Run tests with coverage
          run: |
            mkdir -p EVIDENCE/S08
            pytest -q --junitxml=EVIDENCE/S08/test-report.xml --cov=app --cov-report xml:EVIDENCE/S08/coverage.xml
          continue-on-error: false

        - name: Upload artifacts
          if: always()
          uses: actions/upload-artifact@v4
          with:
            # name: evidence-s08
            name : evidence-s08-py${{ matrix.python-version }}
            path: |
              EVIDENCE/S08/**
            if-no-files-found: warn
            retention-days: 14
          

  ```

- **Стабильность:** 5 запусков зелёные
- **Что сделано:** включены отчёт покрытия, матрица питон-версии, concurrency, кэш pip

---

## 4) Артефакты и логи конвейера (DV4)

_Сложите файлы в `/EVIDENCE/` и подпишите их назначение._


| Артефакт/лог               | Путь в `EVIDENCE/S08`             | Комментарий                                                                       |
| -------------------------- | --------------------------------- | --------------------------------------------------------------------------------- |
| Список артефактов          | `artifacts.txt`      | Сводный список всех файлов-артефактов, созданных в процессе CI.                   |
| Доказательство кеша        | `cache-proof.txt`    | Лог/подтверждение работы механизма кеширования pip или сборки.                    |
| Логи CI (сырые)            | `ci-log.txt`         | Подробный журнал выполнения конвейера CI.                                         |
| Сводка запуска CI          | `ci-run.txt`         | Краткое резюме результатов запуска и статуса джобов.                              |
| Доказательство concurrency | `concurrency.txt`    | Подтверждение настройки `concurrency` (group и cancel).                           |
| Отчёт покрытия кода        | `coverage.xml`       | Отчёт о покрытии кода в формате Cobertura XML, полученный из `pytest-cov`.        |
| Валидация JUnit            | `junit-validate.txt` | Лог проверки корректности формата JUnit-отчёта.                                   |
| Доказательство матрицы     | `matrix-proof.png`   | Скриншот/доказательство параллельного выполнения джобов по разным версиям Python. |
| Игнор путей (proof)        | `paths-ignore.txt`   | Подтверждение корректной работы правила `paths-ignore`.                           |
| Отчёт тестов (JUnit XML)   | `test-report.xml`    | Итоговый отчёт о тестах в формате JUnit для хранения и анализа в CI.              |

---

## 5) Секреты и переменные окружения (DV5 - гигиена, без сканеров)

- **Шаблон окружения:** добавлен файл `/.env.example` со списком переменных (без значений), например:
  - `SECRET_KEY=`
- **Хранение и передача в CI:**  
  - секреты лежат в настройках репозитория/организации (**masked**),  
  - в pipeline они **не печатаются** в явном виде.
- **Пример использования секрета в job:**

  ```yaml
  - name: Mask any incidental value (подстраховка)
    run: echo "::add-mask::${{ secrets.SECRET_KEY }}"
  ```

  _Есть секрет (`EVIDENCE/S08/secrets-proof.txt`), но в лог не печатается._


---

## 6) Индекс артефактов DV

_Чтобы преподаватель быстро сверил файлы._

| Тип     | Файл в `EVIDENCE/`            | Дата/время         | Коммит/версия | Runner/OS    |
|---------|--------------------------------|--------------------|---------------|--------------|
| "Like search before" screenshot  | `S06/like_search_before.png` | 2025-10-21 |   `092801c0dbb40c148a94388b051c738dcd9c4b45`  | `local`|
| "Like search after" screenshot | `S06/like_search_after.png`   |   2025-10-21        |  `e3c13f2fc179d464f1f35a98dd1f79a9ba35047f` | `local`      |
| "XSS after" screenshot  | `S06/xss_after.png`    | 2025-10-21 |  `bc7014a8d43538a5e3dee1bae2f714be202c5654`  | `local`   |
| Build log | `S07/build.log`            | 2025-10-21               | `3f9007fbc645135f7fccfe2e6490fde85d581044`  | `local`  |
| Compose log  | `S07/compose-up.log`  |2025-10-21                | `3f9007fbc645135f7fccfe2e6490fde85d581044`   | `local`    |
| Healthcheck   | `S07/health.json`            | 2025-10-21              | `3f9007fbc645135f7fccfe2e6490fde85d581044`    | `local`     |
| Non-root check   | `S07/non_root.txt`            | 2025-10-21              | `3f9007fbc645135f7fccfe2e6490fde85d581044`    | `local`     |
| HTTP root code   | `S07/http_root_code.txt`            | 2025-10-21              | `3f9007fbc645135f7fccfe2e6490fde85d581044`    | `local`     |
| Inspect web   | `S07/inscpect_web.json`            | 2025-10-21              | `3f9007fbc645135f7fccfe2e6490fde85d581044`    | `local`     |
| Docker Image size   | `S07/image_size.txt`            | 2025-10-21              | `3f9007fbc645135f7fccfe2e6490fde85d581044`    | `local`     |
| Список артефактов          | `S08/artifacts.txt`      |   2025-10-21  | `8471e1db727311901b717f4fd50bd1ac3f98d7c4` | `local` |
| Доказательство кеша        | `S08/cache-proof.txt`    |    2025-10-21    | `8471e1db727311901b717f4fd50bd1ac3f98d7c4` | `local` |
| Логи CI (сырые)            | `S08/ci-log.txt`         |  2025-10-21    | `8471e1db727311901b717f4fd50bd1ac3f98d7c4`| `local`     |
| Доказательство concurrency | `S08/concurrency.txt`    | 2025-10-21 |  `8471e1db727311901b717f4fd50bd1ac3f98d7c4` | `local`     |
| Отчёт покрытия кода        | `S08/coverage.xml`       |  2025-10-21 | `8471e1db727311901b717f4fd50bd1ac3f98d7c4`  | `Github Runner` |
| Валидация JUnit            | `S08/junit-validate.txt` | 2025-10-21  | `8471e1db727311901b717f4fd50bd1ac3f98d7c4`  | `local`     |
| Доказательство матрицы     | `S08/matrix-proof.png`   | 2025-10-21 |   `8471e1db727311901b717f4fd50bd1ac3f98d7c4` | `local`     |
| Игнор путей (proof)        | `S08/paths-ignore.txt`   |  2025-10-21|  `8471e1db727311901b717f4fd50bd1ac3f98d7c4`  | `local`     |
| Отчёт тестов (JUnit XML)   | `S08/test-report.xml`    | 2025-10-21 |   `8471e1db727311901b717f4fd50bd1ac3f98d7c4`      | `Github Runner` |
| Доказательство наличии секрета   | `S08/secret-proof.xml`    | 2025-10-21 |    `8471e1db727311901b717f4fd50bd1ac3f98d7c4`     | `Github Runner` |
---

## 7) Связь с TM и DS (hook)

- **TM:** этот конвейер обслуживает риски процесса сборки/поставки (например, культура работы с секретами, воспроизводимость).  
- **DS:** сканы/гейты/триаж будут оформлены в `DS.md` с артефактами в `EVIDENCE/`.

---

## 8) Самооценка по рубрике DV (0/1/2)

- **DV1. Воспроизводимость локальной сборки и тестов:** [ ] 0 [ ] 1 [x] 2  
- **DV2. Контейнеризация (Docker/Compose):** [ ] 0 [ ] 1 [x] 2  
- **DV3. CI: базовый pipeline и стабильный прогон:** [ ] 0 [ ] 1 [x] 2  
- **DV4. Артефакты и логи конвейера:** [ ] 0 [ ] 1 [x] 2  
- **DV5. Секреты и конфигурация окружения (гигиена):** [ ] 0 [ ] 1 [x] 2  

**Итог DV (сумма):** 10/10
