# Macro Dashboard — САЩ & Еврозона

Интерактивен уеб дашборд за макроикономически анализ на САЩ и Еврозоната.
Хостнат на **GitHub Pages** — достъпен от всяко устройство без инсталация.

🔗 **Live:** https://tsvetoslavtsachev.github.io/macro-web-dashboard/

---

## Архитектура

```
us-macro-dashboard  ──► output/api/macro_state.json  ──┐
                    ──► output/api/series_data.json   ──┤
                                                        ├──► macro-web-dashboard (GitHub Pages)
eu-macro-dashboard  ──► output/api/macro_state.json  ──┤
                    ──► output/api/series_data.json   ──┘
```

Уеб приложението е **статичен HTML файл** (без сървър, без база данни).
При зареждане прави `fetch()` към `raw.githubusercontent.com` и чете JSON файловете директно от двете макро репо.

---

## Секции

| Секция | Описание |
|---|---|
| **Обзор** | Карти за всички лещи — breadth score, посока, аномалии |
| **Трудов пазар** | NFP, безработица, JOLTS, заплати, LFPR и др. |
| **Инфлация** | CPI, PCE, PPI, breakevens, инфлационни очаквания |
| **Растеж** | GDP, PMI, retail sales, industrial production |
| **Ликвидност / Кредит** | Спредове, финансови условия, Fed баланс |
| **Аномалии** | Всички серии с Z-score > 2σ, нови исторически екстремуми |
| **Дивергенции** | Cross-lens сигнали — кои лещи са в конфликт |

---

## Обновяване на данните

Данните се обновяват автоматично чрез GitHub Actions в двете макро репо:

- `us-macro-dashboard` → всяка събота 09:30 Sofia time
- `eu-macro-dashboard` → всяка събота 09:30 Sofia time

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
