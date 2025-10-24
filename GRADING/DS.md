# DS - Отчёт «DevSecOps-сканы и харднинг»

> Этот файл - **индивидуальный**. Его проверяют по **rubric_DS.md** (5 критериев × {0/1/2} → 0-10).
> Подсказки помечены `TODO:` - удалите после заполнения.
> Все доказательства/скрины кладите в **EVIDENCE/** и ссылайтесь на конкретные файлы/якоря.

---

## 0) Мета

- **Проект (опционально BYO):** «учебный шаблон»
- **Версия (commit/date):** b4937f8339625d3891c0096c95235c95d11e482c / 2025-10-24
- **Кратко (1-2 предложения):** сканируются SBOM и уязвимости зависимостей, SAST и Secrets, DAST и Policy и планируются non-root пользователь, HEALTHCHECK харднинга

---

## 1) SBOM и уязвимости зависимостей (DS1)

- **Инструмент/формат:** Syft/Grype; CycloneDX
- **Как запускал:**

  ```bash
  syft dir:. -o cyclonedx-json > EVIDENCE/S09/sbom.json
  grype sbom:/work/EVIDENCE/S09/sbom.json -o json > EVIDENCE/S09/sca_report.json
  ```

- **S09 - SBOM & SCA**: сгенерирован SBOM (CycloneDX), выполнен SCA. 
- **Артефакты**: EVIDENCE/S09/sbom.json, EVIDENCE/S09/sca_report.json. 
- **Сводка**: EVIDENCE/S09/sca_summary.md. 
- **Actions**: https://github.com/phamtienmanh2001/secdev-s09-s12/actions/runs/18738552439
- **Ссылка на успешный job**: https://github.com/phamtienmanh2001/secdev-s09-s12/actions/runs/18738552439

---

## 2) SAST и Secrets (DS2)

### 2.1 SAST

- **Инструмент/профиль:** semgrep
- **Как запускал:**

  ```bash
  semgrep semgrep ci --config p/ci --sarif --output /src/EVIDENCE/S10/semgrep.sarif --metrics=off || true
  ```

- **Артефакты:** `EVIDENCE/S10/semgrep.sarif`
- **Выводы:** FP рассмотрены
- **Ссылка на успешный job**: https://github.com/phamtienmanh2001/secdev-s09-s12/actions/runs/18738552421

### 2.2 Secrets scanning

- **Инструмент:** gitleaks
- **Как запускал:**

  ```bash
  gitleaks detect --source=/repo --report-format=json --report-path=/repo/EVIDENCE/S10/gitleaks.json || true
  ```

- **Артефакты:** `EVIDENCE/S10/gitleaks.json`
- **Ссылка на успешный job**: https://github.com/phamtienmanh2001/secdev-s09-s12/actions/runs/18738552421
---

## 3) DAST и Policy (Container/IaC) (DS3)


### DAST (лайт)

- **Инструмент/таргет:** zap
- **Как запускал:**

  ```bash
  zap-baseline.py -t http://localhost:8080 -r zap_baseline.html -J zap_baseline.json-d || true
  ```

- **Артефакты:** `EVIDENCE/S11/zap_baseline.html`, `EVIDENCE/S11/zap_baseline.json-d.json`
- **Выводы:** В этом проекте есть 2 средних риска, 3 низких риска.
- **Ссылка на успешный job**: https://github.com/phamtienmanh2001/secdev-s09-s12/actions/runs/18777387027

### Policy / Container / IaC

- **Инструмент(ы):** trivy config / checkov 
- **Как запускал:**

  ```bash
  trivy image --format json --output /work/EVIDENCE/S12/trivy.json --ignore-unfixed s09s12-app:ci || true
  checkov -d /src/iac -o json > EVIDENCE/S12/checkov.json || true
  ```

- **Артефакты:** `EVIDENCE/S12/trivy.json`, `EVIDENCE/S12/checkov.json`
- **Ссылка на успешный job**: https://github.com/phamtienmanh2001/secdev-s09-s12/actions/runs/18787129728
---
---

## 4) Харднинг (доказуемый) (DS4)

Отметьте **реально применённые** меры, приложите доказательства из `EVIDENCE/`.

- [x] **Контейнер non-root / drop capabilities** → Evidence: `EVIDENCE/S12/non-root.txt`
- [x] **HEALTHCHECK** → Evidence: `EVIDENCE/S12/health.json`
- [x] **Отказ от latest** -> Evidence: `EVIDENCE/S12/checkov.json`
- [x] **K8s non-root** -> Evidence: `EVIDENCE/S12/checkov.json`

> До - 17 предупреждений (checkov-old.json), после - 15 (checkov.json)


---

## 5) Quality-gates и проверка порогов (DS5)

- **Пороговые правила (словами):**  
  SCA: Critical=0; High≤1, «SAST: Critical=0», «Secrets: 0 истинных находок», «Policy: Violations=0».
- **Как проверяются:**  
  - Ручной просмотр папки EVIDENCE/S09, EVIDENCE/S10, EVIDENCE/S11, EVIDENCE/S12, 

---

## 6) Эффект «до/после» (метрики) (DS4/DS5)

| Контроль/Мера | Метрика                 | До   | После | Evidence (до), (после)                          |
|---------------|-------------------------|-----:|------:|-------------------------------------------------|
| Policy/IaC    | Violations              | 17 | 15    | `EVIDENCE/checkov-old.txt`, `checkov.txt` |


---

## 7) Самооценка по рубрике DS (0/1/2)

- **DS1. SBOM и SCA:** [ ] 0 [ ] 1 [x] 2  
- **DS2. SAST + Secrets:** [ ] 0 [ ] 1 [x] 2  
- **DS3. DAST или Policy (Container/IaC):** [ ] 0 [ ] 1 [x] 2  
- **DS4. Харднинг (доказуемый):** [ ] 0 [ ] 1 [x] 2  
- **DS5. Quality-gates, триаж и «до/после»:** [ ] 0 [ ] 1 [x] 2  

**Итог DS (сумма):** 10/10
