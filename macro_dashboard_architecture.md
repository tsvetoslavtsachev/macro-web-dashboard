# Архитектура на Macro Analytics Dashboard

Този документ описва архитектурата за свързване на съществуващите бекенд репозитории (`us-macro-dashboard` и `eu-macro-dashboard`) с нов статичен фронтенд дашборд.

## 1. Обща архитектура

Системата следва "Static Site Generator" (SSG) / "Data API" модел, базиран изцяло на GitHub инфраструктура:

1.  **Data Providers (Бекенд):**
    *   Репозитории: `us-macro-dashboard` и `eu-macro-dashboard`.
    *   Изпълняват се чрез GitHub Actions (cron job, напр. всяка събота сутрин).
    *   Изтеглят данни от FRED/ECB, изпълняват аналитичния слой (scoring, regimes).
    *   **Ново:** Експортират резултатите в статични JSON файлове в папка `api/` (или `data/`).
    *   Публикуват тези JSON файлове чрез GitHub Pages (или са достъпни през `raw.githubusercontent.com`).

2.  **Frontend Dashboard (Клиент):**
    *   Ново репозиторие: `macro-analytics-dashboard`.
    *   Статичен HTML/JS/CSS (без build стъпка или с прост bundler като Vite, ако е нужно, но предвид предпочитанията за простота - vanilla JS е напълно достатъчен).
    *   Хостнат на GitHub Pages.
    *   При зареждане прави `fetch()` заявки към JSON файловете от Data Providers.
    *   Рендерира UI: табове за региони, секции за лещи (labor, inflation и т.н.), графики (Plotly.js) и текстови анализи.

## 2. JSON Data Schema (Договор между Backend и Frontend)

За да бъде фронтендът бърз и да не прави тежки изчисления, бекендът трябва да подготви данните във формат, готов за визуализация. Предлагам два основни JSON файла за всеки регион (US и EU):

### A. `macro_state.json` (Аналитичен слой)
Този файл съдържа високото ниво на анализа - режими, аномалии, дивергенции. Той е малък и се зарежда първи.

```json
{
  "region": "US",
  "as_of_date": "2026-05-15",
  "executive_summary": {
    "composite_score": 68,
    "regime_label": "soft_landing",
    "regime_label_bg": "Soft landing",
    "css_class": "regime-soft",
    "narrative": [
      "Трудовият пазар остава затегнат, докато инфлацията се охлажда.",
      "Кредитните условия са стабилни."
    ]
  },
  "lenses": {
    "labor": {
      "score": 75,
      "regime": "HEALTHY",
      "direction": "expanding",
      "breadth_pct": 85,
      "anomalies_count": 1,
      "divergences": [
        "Спад в заявените свободни работни места, но безработицата остава ниска."
      ]
    },
    "inflation": {
      "score": 45,
      "regime": "COOLING",
      "direction": "contracting",
      "breadth_pct": 40,
      "anomalies_count": 0,
      "divergences": []
    }
    // ... останалите лещи (growth, credit, housing/ecb)
  },
  "top_anomalies": [
    {
      "series_id": "TRUCK_EMP",
      "name_bg": "Заети: автомобилен транспорт",
      "lens": "labor",
      "z_score": -2.5,
      "current_value": -1.2,
      "transform": "yoy_pct"
    }
  ],
  "cross_lens_divergences": [
    {
      "pair": "labor_vs_credit",
      "description": "Трудовият пазар е силен, но кредитните спредове се разширяват."
    }
  ]
}
```

### B. `series_data.json` (Данни за графиките)
Този файл съдържа времевите редове за ключовите индикатори, нужни за изчертаване на графиките. За да не става прекалено голям, ще експортираме само последните N години (напр. 5 или 10 години) или само сериите, които искаме да визуализираме.

```json
{
  "region": "US",
  "last_updated": "2026-05-15T10:00:00Z",
  "series": {
    "UNRATE": {
      "meta": {
        "name_bg": "Безработица (%)",
        "lens": "labor",
        "transform": "level",
        "is_rate": true
      },
      "data": {
        "dates": ["2023-01-01", "2023-02-01", "...", "2026-04-01"],
        "values": [3.4, 3.6, "...", 3.8]
      },
      "latest": {
        "date": "2026-04-01",
        "value": 3.8,
        "change_yoy": 0.2
      }
    },
    "PAYEMS": {
      "meta": {
        "name_bg": "NFP (общо заети)",
        "lens": "labor",
        "transform": "yoy_pct",
        "is_rate": false
      },
      "data": {
        "dates": ["2023-01-01", "2023-02-01", "...", "2026-04-01"],
        "values": [2.5, 2.3, "...", 1.5]
      },
      "latest": {
        "date": "2026-04-01",
        "value": 1.5,
        "change_yoy": -1.0
      }
    }
    // ... останалите ключови серии
  }
}
```

## 3. Необходими промени в Backend (us-macro-dashboard / eu-macro-dashboard)

1.  **Нов скрипт `export_api.py`:**
    *   Ще импортира съществуващите модули от `analysis/` (както прави `weekly_briefing.py`).
    *   Ще събира данните в речници (dictionaries), съответстващи на JSON схемата по-горе.
    *   Ще записва резултата в `output/api/macro_state.json` и `output/api/series_data.json`.
2.  **Обновяване на GitHub Actions:**
    *   Създаване на workflow (напр. `.github/workflows/update-api.yml`), който:
        *   Изпълнява `python run.py --refresh` (за да дръпне нови данни).
        *   Изпълнява `python export_api.py`.
        *   Къмитва и пушва промените в `output/api/` към `main` бранча (или deploy-ва към `gh-pages` бранч).

## 4. Структура на Frontend (macro-analytics-dashboard)

```text
macro-analytics-dashboard/
├── index.html          # Основен UI (табове, контейнери за графики)
├── css/
│   └── style.css       # Стилизиране (може да заемем класове от weekly_briefing)
├── js/
│   ├── app.js          # Основна логика (routing, tab switching)
│   ├── data.js         # Fetch логика (взимане на JSON от US/EU репата)
│   └── charts.js       # Plotly.js wrappers за рендериране на графиките
└── README.md
```

### Потребителски интерфейс (UI)
*   **Header:** Заглавие и превключвател "САЩ" / "Еврозона".
*   **Executive Summary:** Голям бадж с текущия режим (напр. "Soft landing") и кратко резюме.
*   **Grid с Лещи:** За всяка леща (Трудов пазар, Инфлация и т.н.):
    *   Текущ статус (score, посока).
    *   Списък с аномалии/дивергенции.
    *   1-2 ключови графики (напр. Безработица + NFP за Трудов пазар).

## 5. Следващи стъпки за изпълнение

1.  **Одобрение на схемата:** Потвърждение, че JSON структурата покрива нуждите.
2.  **Имплементация в Backend:** Написване на `export_api.py` за `us-macro-dashboard` (като proof of concept).
3.  **Имплементация на Frontend:** Създаване на базовия HTML/JS, който да чете генерирания JSON.
4.  **Скалиране:** Прилагане на същия подход за `eu-macro-dashboard`.
