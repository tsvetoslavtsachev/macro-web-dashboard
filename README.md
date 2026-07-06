# Macro Dashboard — САЩ, Еврозона & Китай

Интерактивен уеб дашборд за макроикономически анализ на САЩ, Еврозоната и Китай.
Хостнат на **GitHub Pages** — достъпен от всяко устройство без инсталация.

🔗 **Live:** https://tsvetoslavtsachev.github.io/macro-web-dashboard/

---

## Архитектура

```
us-macro-dashboard     ──► output/api/macro_state.json  ──┐
                       ──► output/api/series_data.json   ──┤
eu-macro-dashboard     ──► output/api/macro_state.json  ──┼──► macro-web-dashboard (GitHub Pages)
                       ──► output/api/series_data.json   ──┤
china-macro-dashboard  ──► output/api/macro_state.json  ──┤
                       ──► output/api/series_data.json   ──┘
```

Уеб приложението е **статичен HTML файл** (без сървър, без база данни).
При зареждане прави `fetch()` към `raw.githubusercontent.com` и чете JSON файловете директно от трите макро репо (табове САЩ / Еврозона / Китай).

---

## Секции

Лещите се показват според региона — навигацията следва данните в JSON-а
(S14 lens contract), не статичен списък.

| Секция | Описание | Региони |
|---|---|---|
| **Обзор** | Карти за всички лещи — breadth score, посока, аномалии | всички |
| **Трудов пазар** | NFP, безработица, JOLTS, заплати, LFPR и др. | US · EA · CN |
| **Инфлация** | CPI, PCE, PPI, breakevens, инфлационни очаквания | US · EA · CN |
| **Растеж** | GDP, PMI, retail sales, industrial production | US · EA · CN |
| **Ликвидност** | Спредове, финансови условия, Fed баланс | US |
| **Кредит & Финансови условия** | Кредитни спредове, банково кредитиране | EA · CN |
| **Външен сектор** | Търговски баланс, външно търсене (EU F-teardown) | EA |
| **Имоти и търговия** | Китайска леща — имотен пазар + търговия | CN |
| **Аномалии** | Всички серии с Z-score > 2σ, нови исторически екстремуми | всички |
| **Дивергенции** | Cross-lens сигнали — кои лещи са в конфликт | всички |

---

## Обновяване на данните

Данните се обновяват автоматично чрез GitHub Actions в трите макро репо:

- `us-macro-dashboard` → седмичен cron
- `eu-macro-dashboard` → седмичен cron
- `china-macro-dashboard` → седмичен cron

За ръчно обновяване:
```bash
# В us-macro-dashboard или eu-macro-dashboard
python export_api.py --refresh --years 7
git add output/api/
git commit -m "chore: update API JSON"
git push
```

---

## Свързани репо

- [us-macro-dashboard](https://github.com/tsvetoslavtsachev/us-macro-dashboard) — данни и анализ за САЩ (FRED API)
- [eu-macro-dashboard](https://github.com/tsvetoslavtsachev/eu-macro-dashboard) — данни и анализ за Еврозоната (ECB/Eurostat)
- [china-macro-dashboard](https://github.com/tsvetoslavtsachev/china-macro-dashboard) — данни и анализ за Китай
