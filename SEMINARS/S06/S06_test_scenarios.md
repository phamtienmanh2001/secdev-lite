# S06 - Test Scenarios Template

> Цель: быстро оформить 4-6 автотестов, которые подтверждают ваши фиксы из `S06_patch_cards.md`, а также **пишут отчёт** в `EVIDENCE/S06/test-report.xml` для DV.

## Как этим пользоваться

1. Выберите **минимум 2 карточки** из `S06_patch_cards.md`.
2. На каждую карточку добавьте **≥1 автотест** (лучше 2: позитив + негатив).
3. Прогоняйте локально одной командой (**one-liner** для DV1).
4. Складывайте отчёты и логи в `EVIDENCE/S06/` и дайте ссылки в `GRADING/DV.md`.

---

## Нотация сценариев

Выберите удобную форму - **Given-When-Then** или **pytest-сценарий**. Внутри репозитория тесты реализуются на pytest.

**Шаблон GWT (заполняйте под себя):**

```Gherkin
Сценарий: <краткое имя>
Given:   <исходное состояние/данные/пользователь>
When:    <действие/запрос>
Then:    <ожидаемый результат/код/фрагмент ответа/заголовки>
Artifacts: EVIDENCE/S06/<путь к отчёту/скрину/логу>
Связано с: DV1/DV3/DV4 (указать релевантные)
```

**Шаблон pytest (скелет):**

```python
import pytest
from fastapi.testclient import TestClient
from app.main import app

client = TestClient(app)

def test_<name>():
    # Arrange / Given
    # ТУТ НУЖНА ТВОЯ ВСТАВКА (инициализация/данные)
    # Act / When
    resp = client.get("/ТУТ НУЖНА ТВОЯ ВСТАВКА")
    # Assert / Then
    assert resp.status_code == 200
    # ТУТ НУЖНА ТВОЯ ВСТАВКА (конкретные проверки)
```

---

## Обязательный минимум (под seed FastAPI + SQLite + Jinja2)

Добавьте тесты так, чтобы они подтверждали фиксы. Ниже - заготовки под каждую карточку.

### 1) SQL Injection (login) - обязателен, если брали S06-01

**GWT**

```Gherkin
Сценарий: Логин не уязвим к SQLi-комментариюs
Given:   Пользователь "admin" в БД
When:    POST /login с username="admin'-- " и любым password
Then:    401 Unauthorized
Artifacts: EVIDENCE/S06/test-report.xml
Связано с: DV3, DV4
```

**pytest**

```python
def test_login_should_not_allow_sql_injection():
    payload = {"username": "admin'-- ", "password": "x"}
    resp = client.post("/login", json=payload)
    assert resp.status_code == 401, "SQLi-бэйпас логина должен быть закрыт"
```

**GWT**

```Gherkin
Сценарий: Логин и пароль правильные
Given:   Пользователь "admin" с паролем "admin "в БД
When:    POST /login с username="admin" и password="admin"
Then:    200 OK
Artifacts: EVIDENCE/S06/test-report.xml
Связано с: DV3, DV4
```

**pytest**

```python
def test_login_should_positive():
    payload = {"username": "admin", "password": "admin"}
    resp = client.post("/login", json=payload)
    assert resp.status_code == 200, "Неправильные логин и пароль"
```

### 2) SQL Injection (search LIKE) - если брали S06-02

**GWT**

```Gherkin
Сценарий: q очень длинный -> 400/422
Given:   Предзаполненная таблица items
When:    GET /search?q=x (x есть 40 z)
Then:    400 Bad Request or 422 Unprocessable Entity
Artifacts: EVIDENCE/S06/test-report.xml
Связано с: DV3, DV4
```

**pytest**

```python
def def test_search_long_parameter():
    search_param = "z" * 40
    resp_noise = client.get("/search", params={"q": search_param})
    assert resp_noise.status_code in {422, 400} , "Длина q должна быть ограничена"
```

**GWT**

```Gherkin
Сценарий: Инъекция в LIKE не возвращает все записи
Given:   Предзаполненная таблица items
When:    GET /search?q=' OR '1'='1
Then:    Количество результатов не больше, чем для "шумового" запроса ("zzzzzz")
Artifacts: EVIDENCE/S06/test-report.xml
Связано с: DV3, DV4
```

**pytest**

```python
def def test_search_should_not_return_all_on_injection():
    resp_noise = client.get("/search", params={"q": "zzzzzzzzz"}).json()
    inj = client.get("/search", params={"q": "' OR '1'='1"}).json()
    assert len(inj["items"]) <= len(resp_noise["items"]), "Инъекция в LIKE не должна приводить к выдаче всех элементов"
```

### 3) XSS (экранирование вывода) - если брали S06-03

**GWT**

```Gherkin
Сценарий: Отражённая XSS не исполняется
Given:   Запуск /echo
When:    GET /echo?msg=<script>alert(1)</script>
Then:    В ответе нет тега <script>, контент экранирован
Artifacts: EVIDENCE/S06/test-report.xml; при желании - скрин ответа
Связано с: DV3, DV4
```

**pytest**

```python
def test_echo_escapes_script_tags():
    resp = client.get("/echo", params={"msg": "<script>alert(1)</script>"})
    assert "<script>" not in resp.text
```

**GWT**

```Gherkin
Сценарий: Отражённая XSS не исполняется
Given:   Запуск /echo
When:    GET /echo?msg=<img src=x onerror=alert("pwn")>
Then:    В ответе нет тега <img> и <onerror>, контент экранирован
Artifacts: EVIDENCE/S06/test-report.xml; при желании - скрин ответа
Связано с: DV3, DV4
```

**pytest**

```python
def test_echo_should_escape_img_onerror():
    resp = client.get("/echo", params={"msg": '<img src=x onerror=alert("pwn")>'})
    assert "<img>" not in resp.text and "<onerror>" not in resp.text, "Нельзя рендерить теги/атрибуты, которые могут выполнить JS"
```

### 4) Валидация ввода - если брали S06-04

**GWT**

```Gherkin
Сценарий: Слишком длинный логин отклоняется
Given:   Ограничения модели (например 3..48 символов)
When:    POST /login с username длиной > 200
Then:    422 Unprocessable Entity
Artifacts: EVIDENCE/S06/test-report.xml
Связано с: DV3, DV4
```

**pytest**

```python
def test_login_long():
    payload = {"username": "adminadminadminadminadminadminadminadminadminadmin", "password": "admin"}
    resp = client.post("/login", json=payload)
    assert resp.status_code == 422 , "Длина username and password должна быть ограничена"
```


---

## Как запускать и куда писать отчёт

**Рекомендуемая команда (пример для DV1):**

```bash
pytest -q --junitxml=EVIDENCE/S06/test-report.xml
```

* ТУТ НУЖНА ТВОЯ ВСТАВКА: вы можете упаковать это в `make ci`/скрипт и использовать как **one-liner**.
* При необходимости скриншоты и логи кладите в:

  * `EVIDENCE/S06/screenshots/…`
  * `EVIDENCE/S06/logs/…`
  * `EVIDENCE/S06/patches/*.diff`

---

## Что сослать в GRADING/DV.md

* **DV1 (one-liner):** точную команду (или `make ci`) + ОС/версию языка.
* **DV3 (фиксы кода):** список карточек, коротко «что было → что сделали → где тест».
* **DV4 (артефакты):** пути к `EVIDENCE/S06/test-report.xml`, `logs/*`, `screenshots/*`.
* **DV5 (инструкции):** 2-5 строк «Локальный запуск» в `README` с ссылкой на one-liner.

---

## Definition of Done (чек перед сдачей)

* [ ] Есть **≥4 теста** (лучше 6), минимум по выбранным карточкам.
* [ ] Тесты **зелёные** после фиксов.
* [ ] Отчёт JUnit лежит в `EVIDENCE/S06/test-report.xml`.
* [ ] В `GRADING/DV.md` - ссылки на артефакты и краткие комментарии по фиксам.
* [ ] Секреты не закоммичены; `.env.example` присутствует.

---
