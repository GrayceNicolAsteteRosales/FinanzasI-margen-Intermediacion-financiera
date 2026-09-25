# Base de datos y código — Finanzas I (055D), Unidad I, 2026-II

| Dato | Detalle |
|---|---|
| Nombres y apellidos | ASTETE ROSALES GRAYCE NICOL |
| Código de matrícula | 2024200482A |
| Tema del temario | N.º 2 — El margen de intermediación financiera en el Perú: spread TAMN-TIPMN y sus determinantes |
| Periodo de la base | 2021-01-04 a 2025-12-31 (1245 días hábiles) |
| Ventana de consulta | `FECHA_INICIO = 2021-01-01` · `FECHA_CORTE = 2025-12-31` |
| Fecha de extracción | API BCRP: 2026-09-25 · Scraping SBS: 2026-09-25 |
| Repositorio GitHub | https://github.com/GrayceNicolAsteteRosales/FinanzasI-margen-Intermediacion-financiera |

## 1. Fuentes y vías de extracción

**Vía 1 — API (`01_extraccion_api`).** BCRPData, Banco Central de Reserva del Perú. API REST pública, sin clave.
Endpoint: `https://estadisticas.bcrp.gob.pe/estadisticas/series/api/{codigo}/json/{inicio}/{fin}/esp`
Series: PD12301MD (tasa de referencia), PD04692MD (tasa interbancaria MN), PD04709XD (EMBIG Perú), PD04638PD (tipo de cambio venta).

**Vía 2 — Web scraping (`02_scraping_web`).** Portal de estadísticas de la SBS. La TAMN y la TIPMN diarias no están disponibles por API (BCRPData solo las publica en frecuencia mensual). El script envía el formulario ASP.NET de cada página con la fecha de consulta, con pausa de 1,5 s entre solicitudes y User-Agent identificable, tras revisar el robots.txt.
- TAMN: https://www.sbs.gob.pe/app/pp/EstadisticasSAEEPortal/Paginas/TIActivaMercado.aspx?tip=B
- TIPMN: https://www.sbs.gob.pe/app/pp/EstadisticasSAEEPortal/Paginas/TIPasivaMercado.aspx?tip=B

## 2. Orden de ejecución (Google Colab)

1. `codigo/01_extraccion_api.ipynb` → `datos_crudos/datos_crudos_api_2024200482A.csv`, `datos_crudos/json_bcrp/`
2. `codigo/02_scraping_web.ipynb` → `datos_crudos/datos_crudos_sbs_2024200482A.csv`, `datos_crudos/html_sbs/` y la tabla única `datos_crudos/datos_crudos_2024200482A.csv`
3. `codigo/03_limpieza_datos.ipynb` → `datos_procesados/datos_procesados_2024200482A.csv`, `salidas/hash_sha256.txt`
4. `codigo/04_analisis.ipynb` → tablas y figuras en `salidas/`

Cada cuaderno monta Google Drive y localiza la carpeta del proyecto por su contenido (rutas relativas, sin rutas personales). No se requieren claves de API (ver `.env.example`).

## 3. Entorno

Python 3.13.15 (Google Colab). Librerías:
- requests==2.32.4
- pandas==2.2.3
- numpy==2.1.3
- beautifulsoup4==4.13.5
- lxml==6.1.2
- openpyxl==3.1.5
- matplotlib==3.10.0
- scipy==1.16.3
- statsmodels==0.15.0

## 4. Hash SHA-256 del archivo procesado

```
0b57bc94ff94f1798cb40c2fd8bb266046345662631505e4917a56de2e1f8448  datos_procesados_2024200482A.csv
```

## 5. Tratamiento de los datos

Se eliminaron los feriados peruanos (días sin tasa interbancaria ni tasa de referencia publicadas) y los días con alguna variable faltante. No se imputó ningún valor, de modo que cada observación corresponde a un dato publicado por la fuente oficial. Los valores atípicos se reportan en `salidas/reporte_calidad.csv`, pero no se eliminan. No hay procesos aleatorios, por lo que no se fija semilla.

## 6. Variables

| Variable | Definición | Unidad | Fuente |
|---|---|---|---|
| `id` | Número correlativo de la observación | — | Generado en 03_limpieza_datos |
| `fecha` | Día hábil con información para todas las variables (llave común) | AAAA-MM-DD | — |
| `tamn_pct` | Tasa activa promedio de mercado en moneda nacional (TAMN) | % efectivo anual | Superintendencia de Banca, Seguros y AFP (SBS), web scraping |
| `tipmn_pct` | Tasa pasiva promedio de mercado en moneda nacional (TIPMN) | % efectivo anual | Superintendencia de Banca, Seguros y AFP (SBS), web scraping |
| `spread_pct` | Spread de intermediación: tamn_pct − tipmn_pct (variable dependiente) | puntos porcentuales | Cálculo propio en 03_limpieza_datos |
| `tasa_referencia_pct` | Tasa de referencia de la política monetaria | % anual | Banco Central de Reserva del Perú (BCRP), BCRPData, serie PD12301MD |
| `tasa_interbancaria_pct` | Tasa de interés interbancaria en moneda nacional | % anual | BCRP, BCRPData, serie PD04692MD |
| `brecha_interbancaria_pct` | tasa_interbancaria_pct − tasa_referencia_pct (presión de liquidez) | puntos porcentuales | Cálculo propio en 03_limpieza_datos |
| `riesgo_pais_embig_pct` | Spread EMBIG Perú (riesgo país), convertido de pbs a %: pbs / 100 | % | BCRP, BCRPData, serie PD04709XD |
| `tipo_cambio_pen_usd` | Tipo de cambio interbancario, venta | soles por dólar | BCRP, BCRPData, serie PD04638PD |

Detalle completo en `diccionario_variables.xlsx`.

## 7. Estructura de la carpeta

```
codigo/            01_extraccion_api · 02_scraping_web · 03_limpieza_datos · 04_analisis
datos_crudos/      json_bcrp/ · html_sbs/ · avance_scraping_sbs.csv · datos_crudos_*.csv / .xlsx
datos_procesados/  datos_procesados_2024200482A.csv / .xlsx
salidas/           tablas (.xlsx, .csv, .tex) · figuras (.png, .pdf) · reporte_calidad.csv · hash_sha256.txt
diccionario_variables.xlsx · README.md · requirements.txt · .env.example · log_ejecucion.txt
```

## 8. Citas de las fuentes de datos (APA 7)

Banco Central de Reserva del Perú. (2026). *BCRPData: Series estadísticas diarias* [Base de datos]. Consultado el 25 de septiembre de 2026. https://estadisticas.bcrp.gob.pe/estadisticas/series/

Superintendencia de Banca, Seguros y AFP. (2026). *Tasas de interés promedio del sistema bancario* [Base de datos]. Consultado el 25 de septiembre de 2026. https://www.sbs.gob.pe/app/pp/EstadisticasSAEEPortal/Paginas/TIActivaMercado.aspx?tip=B

## 9. Uso de inteligencia artificial

Se utilizó IA generativa como apoyo para escribir y depurar el código. Cada bloque fue revisado y puede ser explicado por la autora (numeral 2.1 de la consigna).
