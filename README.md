# Corporate Credit Scoring & Rating Model (PD)

Modelo de **probabilidad de quiebra (PD) a 12 meses, score y rating** para empresas cotizadas de EE. UU., con scorecard
WOE + regresión logística, escala maestra de ratings (AAA–C), validación fuera de tiempo y benchmark frente al Z-score de Altman.

**Dataset:** American Companies Bankruptcy (Lombardo et al., 2022) – ~78.000 observaciones empresa-año, 1999-2018, 18 variables contables.
Partición temporal: train 1999-2011 · validación 2012-2014 · test 2015-2018.

## Resultados (test 2015-2018)
> Rellena esta tabla con la salida de `python run_american.py` sobre el CSV real (`outputs_american/metricas.csv`).

| Métrica | Modelo scorecard | Altman Z'' |
|---|---|---|
| AUC | _…_ | _…_ |
| Gini | _…_ | _…_ |
| KS | _…_ | – |
| PSI (train vs test) | _…_ | – |

![validación](outputs_american/validacion_american.png)

## Metodología
1. **Ingeniería de variables:** 18 ratios (rentabilidad, apalancamiento, liquidez, actividad, tamaño) desde los campos X1–X18.
2. **Scorecard:** binning por cuantiles con bin de missing → WOE/IV → filtro IV ≥ 0,02 y correlación < 0,7 → regresión logística L2.
3. **Score:** `Offset + Factor·ln((1-PD)/PD)` (600 pts, odds 50:1, PDO 40).
4. **Escala maestra:** 9 ratings con PD central espaciada geométricamente y calibrada a la cartera (`build_master_scale`).
5. **Validación:** AUC/Gini/KS/Brier, PSI, backtest por rating con test binomial, monotonía, comparación con Altman Z''.
6. **Tests unitarios** de propiedades básicas (PD en (0,1), score decreciente, ratings monótonos, PSI).

## Uso
```bash
pip install -r requirements.txt
# 1) Descarga el CSV y guárdalo en data/american_bankruptcy.csv
python run_american.py            # dataset real
python run_american.py --mock     # prueba con datos simulados (mismo esquema)
python run_pipeline.py            # demo original con datos 100% sintéticos
```

## Limitaciones (honestidad metodológica)
- La etiqueta es **quiebra (Chapter 7/11)**, una aproximación al default bancario.
- Empresas cotizadas grandes: no extrapolable a pymes ni a empresas no cotizadas.
- Varias observaciones por empresa; la partición temporal mitiga, pero no elimina, la dependencia entre observaciones.
- Sin LGD/EAD: no se estima pérdida esperada.

## Estructura
`src/` (data, features, scorecard, validation) · `run_american.py` · `run_pipeline.py` · `tests/`
