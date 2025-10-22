# S06 - Secure Coding Checklist (template)

> Цель: зафиксировать **что именно** вы починили на S06, **где это подтверждено тестами/артефактами**, и **как это мапится на DV1-DV5**.

## Как пользоваться

1. Отметьте **минимум 2 карточки** из `S06_patch_cards.md`, которые вы реализуете.
2. В каждом разделе ниже проставьте чекбоксы, укажите **коммиты** и **пути к пруфам** в `EVIDENCE/S06/`.
3. В конце заполните сводку для переноса в `GRADING/DV.md`.

---

## Паспорт выполнения (заполнить)

* Команда/репозиторий: https://github.com/phamtienmanh2001/secdev-s06-s08
* Коммит(ы) с фиксом:  (короткие SHA)
* One-liner (DV1): python -m venv .venv; .\.venv\Scripts\Activate.ps1; pip install -r requirements.txt; python scripts/init_db.py; pytest -q --junitxml=EVIDENCE/S06/test-report.xml

  Отчёт тестов: `EVIDENCE/S06/test-report.xml` (или `.txt/.html`)
* Кратко «что было → что сделали»: 

  * строковая интерполяция f"...'{payload.username}'..." -> заменили на параметризованный запрос с плейсхолдерами и передачей кортежа параметров

  * формирование SQL через конкатенацию/интерполяцию (... LIKE '%{q}%') -> перешли на параметризованный запрос с плейсхолдером (WHERE name LIKE ?) и передачей pattern = f"%{q}%" в параметры

  * Добавили валидацию запроса через Query(min_length=1, max_length=32), чтобы ограничить длину входа и отвергнуть пустые/слишком длинные строки.

  * в шаблоне использовался {{ message|safe }} — принудительное отключение экранирования Jinja2; пользовательский ввод вставлялся в HTML как есть -> убрали |safe, теперь {{ message }} рендерится с автоэкранированием Jinja2; вредоносные теги будут отображаться как текст, а не выполняться.

  * поля username и password — простые str, без ограничений длины, пробелов или формата -> добавили валидацию длины (min_length, max_length), очистку пробелов (strip_whitespace=True) и паттерн допустимых символов для логина.

---

## Выбранные карточки (отметьте не менее двух)

* [x] **S06-01 - SQLi (login)**
* [x] **S06-02 - SQLi (search LIKE)**
* [x] **S06-03 - XSS (экранирование вывода)**
* [x] **S06-04 - Валидация ввода (строгость/длины/паттерны)**
* [ ] **S06-05 - Ошибки и логи без утечек**
* [ ] **S06-06 - Security-заголовки (минимум)**
* [ ] **S06-07 - Гигиена секретов и конфигов**
* [ ] **S06-08 - Anti-bruteforce (лайт)**

---

## 1) SQL Injection - login (`POST /login`)

* [x] Параметризация запроса (никаких f-строк)
* [x] Тест(ы) подтверждают блокировку `admin'--`
* [x] (Опц.) Ограничения на логин/пароль в модели

  **Пруфы:** `EVIDENCE/S06/test-report.xml`

  **Коммиты:** 54dbeb196dba25b14936abdcc03710303b528eab

  **Комментарий:** Fix login SQL injection and add positive test login

---

## 2) SQL Injection - search (`GET /search?q=...`)

* [x] Параметризация `LIKE ?` + шаблон `%q%`
* [x] Тест(ы) подтверждают, что инъекция не возвращает все записи
* [x] (Опц.) Лимит длины `q`

  **Пруфы:** `EVIDENCE/S06/test-report.xml`, скрин «до/после» (опц.)

  **Коммиты:** f6dd8e54a2033795fcdfdc5a850918333e27c7c1
  
  **Комментарий:** Fix search SQL injection and add test about length for searching

---

## 3) XSS - экранирование вывода (шаблоны Jinja2)

* [x] Убрано `|safe` или внедрено безопасное кодирование
* [x] Тест(ы) подтверждают отсутствие `<script>` в ответе
  **Пруфы:** `EVIDENCE/S06/test-report.xml`, `EVIDENCE/S06/screenshots/xss_before_after.png` (опц.)

  **Коммиты:** bc7014a8d43538a5e3dee1bae2f714be202c5654

  **Комментарий:** Fix XSS safe and add test img onerror

---

## 4) Валидация ввода (строгость)

* [x] Модель `LoginRequest` ужата по длинам/паттернам
* [x] `q` в `/search` ограничен (min/max)
* [x] Тест(ы) на 400/422 и на корректные значения

  **Пруфы:** `EVIDENCE/S06/test-report.xml`
  **Коммиты:** e3c13f2fc179d464f1f35a98dd1f79a9ba35047f
  **Комментарий:** Fix validation input and add test long input for login

---


## Артефакты S06 (минимум)

* [x] `EVIDENCE/S06/test-report.xml` (или `.txt/.html`)
* [ ] `EVIDENCE/S06/logs/*` (если есть)
* [ ] `EVIDENCE/S06/screenshots/*` (если есть)
* [ ] `EVIDENCE/S06/patches/*.diff` (опц.)

---

## Сводка для переноса в `GRADING/DV.md`

* **DV1 (Воспроизводимость):** one-liner → python -m venv .venv; .\.venv\Scripts\Activate.ps1; pip install -r requirements.txt; python scripts/init_db.py; pytest -q --junitxml=EVIDENCE/S06/test-report.xml, отчёт → `EVIDENCE/S06/test-report.xml`
* **DV2 (Гигиена секретов):** `.env.example`, отсутствие секретов в репо, чтение через env
* **DV3 (Минимальная безопасность кода):** карточки: **ТУТ НУЖНА ТВОЯ ВСТАВКА** (список), кратко «что было → что сделали»
* **DV4 (Артефакты/логи):** перечислите пути: **ТУТ НУЖНА ТВОЯ ВСТАВКА**
* **DV5 (Инструкции):** ссылка на README-раздел «Локальный запуск» + отсылка к one-liner

---

## Definition of Done - перед сдачей

* [x] Выбраны **≥2** карточки, все тесты по ним **зелёные**
* [x] Коммиты/диффы понятны; ссылки на артефакты рабочие
* [x] One-liner без интерактива; отчёт в `EVIDENCE/S06/`
* [x] Секретов нет; `.env.example` присутствует
