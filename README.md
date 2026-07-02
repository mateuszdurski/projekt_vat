# Uszczelnienie VAT a dochody budżetowe Polski 1999–2025

Analiza ilościowa efektu reform fiskalnych JPK (2016), Split Payment (2018)
i Białej Listy (2019) na efektywność poboru VAT w Polsce.

📊 **[Interaktywny raport HTML](https://github.com/mateuszdurski/projekt_vat/blob/main/raport.html)**

---

## Pytania badawcze

1. Czy reformy 2016–2019 zwiększyły efektywność poboru VAT?
2. Czy istnieje statystycznie istotny przełom strukturalny po 2016?
3. O ile wzrosły realne wpływy VAT względem trendu bazowego?
4. Czy wzrost ma charakter trwały czy cykliczny?

---

## Wyniki

| Pytanie | Wynik | Metoda |
|---|---|---|
| Efektywność poboru | **TAK — +0,49 pp** | Test t, Cohen's d=0,86 |
| Structural break | **TAK — p=0,008** | OLS AR, JPK_2016 +0,33 pp |
| Skala efektu | **~684 mld PLN** | Analiza counterfactual |
| Trwałość wzrostu | **TRWAŁY** | HP Filter, Δ trend +0,42 pp |

---

## Dane

| Źródło | Zmienna | Zakres | Format |
|---|---|---|---|
| Ministerstwo Finansów | VAT, CIT, Akcyza (mld PLN) | 1999Q1–2025Q4 | XLS (5 formatów) |
| Eurostat `namq_10_gdp` | PKB kwartalne (CP_MNAC, SCA) | 1999Q1–2025Q4 | API JSON |

Dane MF pobierane z plików `sprawozdanie_operatywne_12_[rok].xls`,
arkusz TABLICA 3. Dane narastające różnicowane do wartości kwartalnych.
ETL obsługuje automatycznie 5 różnych formatów plików (1999, 2000–2003,
2004–2015, 2016–2017, 2018–2025).

> **Uwaga:** Pliki surowe (`raw/`) nie są załączone w repozytorium
> ze względu na rozmiar. Można je pobrać ze strony
> [Ministerstwa Finansów](https://www.gov.pl/web/finanse/sprawozdania-operatywne).

---

## Struktura projektu

```
projekt_vat/
│
├── notebooks/
│   ├── 01_etl.ipynb                  — ETL: wczytanie danych MF + Eurostat
│   ├── 02_eda.ipynb                  — Eksploracyjna analiza danych
│   ├── 03_analiza_statystyczna.ipynb — Testy statystyczne + model OLS AR
│   ├── 04_counterfactual.ipynb       — Analiza counterfactual
│   └── 05_hp_filter.ipynb            — Dekompozycja HP Filter
│
├── processed/
│   ├── df.parquet                    — Dane kwartalne (n=108)
│   └── df_rok.parquet                — Dane roczne (n=26)
│
├── html/
│   ├── eda_vat_interaktywny.html
│   ├── boxplot.html
│   ├── heatmapa_korelacji.html
│   ├── scatter_vat_pkb.html
│   ├── statystyki_opisowe.html
│   ├── counterfactual.html
│   └── hp_filter.html
│
├── raw/                              — Pliki XLS MF (w .gitignore)
├── raport.html                       — Finalny raport ze storytellingiem
├── .gitignore
└── README.md
```

---

## Metodologia

### ETL
- Automatyczne wykrywanie i parsowanie 5 formatów plików XLS
- Dane narastające → różnicowanie do wartości kwartalnych
- Połączenie z danymi PKB z API Eurostatu
- Zapis do formatu Parquet

### EDA
- Dashboard 6-panelowy (Plotly)
- Boxplot, heatmapa korelacji Spearmana, scatter VAT vs PKB
- Statystyki opisowe w podziale przed/po reformach

### Analiza statystyczna
```
Kolejność testów:
1. ADF (stacjonarność)       → p=0,000 ✅ — OLS na poziomach uzasadniony
2. Shapiro-Wilk (normalność) → p=0,94 / 0,76 ✅ — test t parametryczny
3. Levene (wariancje)        → p=0,177 ✅ — Student t-test
4. Test t                    → p<0,001 ✅ — istotna różnica średnich
5. Cohen's d                 → d=0,86 ✅ — duży efekt ekonomiczny
6. Test F (SPLIT + WLIST)    → p=0,942 — usunięte z modelu finalnego
```

**Model finalny OLS AR:**
```
vat_pkb_pct = α + AR(1) + AR(4) + Q1 + Q2 + Q3 + JPK_2016 + COVID_2020 + ε
HAC Newey-West SE, maxlags=4
```
- n=104, R²=0,366, Adj.R²=0,319
- DW=1,955 ✅ · Ljung-Box lag=4 p=0,100 ✅ · JB p=0,070 ✅

### Counterfactual
- Model bazowy AR estymowany na 1999Q1–2016Q2 (n=70)
- Rekurencyjna prognoza na 2016Q3–2025Q4 (n=38)
- Luka = rzeczywistość − scenariusz bez reform

### HP Filter
- Parametr λ=1600 (standard kwartalny, Hodrick-Prescott 1997)
- Robustness check: λ ∈ {400, 1600, 6400, 25600}
- Δ trend = +0,37 do +0,45 pp — wynik stabilny

---

## Uruchomienie

### Wymagania
```bash
pip install pandas numpy statsmodels plotly scipy pyarrow \
            xlrd openpyxl requests jupyter
```

### Kolejność uruchamiania notebooków
```bash
jupyter notebook notebooks/01_etl.ipynb                   # ETL — wymaga plików raw/
jupyter notebook notebooks/02_eda.ipynb                   # EDA
jupyter notebook notebooks/03_analiza_statystyczna.ipynb  # Testy statystyczne i budowa modelu
jupyter notebook notebooks/04_counterfactual.ipynb      
jupyter notebook notebooks/05_hp_filter.ipynb
```

> Notebooki 02–05 wczytują dane z `processed/df.parquet`
> i `processed/df_rok.parquet` — nie wymagają ponownego uruchamiania ETL.

---

## Ograniczenia

- **Korelacja ≠ przyczynowość** — nie można wykluczyć wpływu
  innych czynników (wzrost płac, cyfryzacja, koniunktura)
- **Brak grupy kontrolnej** — wzmocnienie analizy wymagałoby
  podejścia difference-in-differences z krajami V4
- **Counterfactual ~684 mld PLN jest orientacyjny** —
  niepewność prognozy rośnie z horyzontem, CI po 2020 bardzo szerokie
- **HP Filter end-of-sample bias** — wyniki dla 2024–2025
  należy interpretować ostrożniej

---

## Technologie

![Python](https://img.shields.io/badge/Python-3.12-blue)
![pandas](https://img.shields.io/badge/pandas-2.x-green)
![statsmodels](https://img.shields.io/badge/statsmodels-0.14-orange)
![Plotly](https://img.shields.io/badge/Plotly-5.x-purple)
![Jupyter](https://img.shields.io/badge/Jupyter-notebook-orange)

---

## Autor

**Mateusz Durski** · 2026

[![LinkedIn](https://img.shields.io/badge/LinkedIn-Mateusz_Durski-blue)](https://www.linkedin.com/in/mateusz-durski/)
[![GitHub](https://img.shields.io/badge/GitHub-MateuszDurski-black)](https://github.com/MateuszDurski)
