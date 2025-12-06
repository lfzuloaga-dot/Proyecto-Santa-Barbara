
# Evaluación Financiera del Taller Santa Bárbara

## Propósito

Modelar la viabilidad financiera del taller **Santa Bárbara** mediante simulación de escenarios (optimista, base y pesimista). Se proyectan ingresos, costos, tributos, inversión inicial y flujos mensuales; además, se calculan **TIR**, **VAN**, **VPN (WACC)** y el **punto de equilibrio**.

## Índice

* Visión General
* Fundamentos Matemáticos
* Parámetros del Modelo
* Pseudocódigo
* Suposiciones y Alcances
* Validación y Calidad de Resultados
* Reproducibilidad
* Extensiones Sugeridas


## Visión General
El algoritmo ejecuta una simulación determinística por **mes** y por **escenario**, incorporando:
- Inflación y crecimiento real de ingresos.
- Variación del tipo de cambio (Bs/USD).
- Estructura de costos con **dolarización**.
- Tributos (Actividad Económica) e **ISLR** por tramos.

**Salidas principales:**
* Tabla mensual: ingresos, costos, tributos, inversión, ISLR, flujo neto y acumulado.
* Mes de **punto de equilibrio** (primer mes con flujo acumulado positivo).
* Indicadores financieros: **TIR**, **VAN** (suma de flujos) y **VPN** (descontado con WACC).

---

## Fundamentos Matemáticos

**Actualización mensual de índices**
\[
f_{\text{infl}}(i) = (1+\text{inflación})^i,\qquad
tc_{i} = tc_{\text{inicial}}\cdot(1+\text{crec\_tc})^i
\]

**Ingresos proyectados**
\[
\text{Ingreso}_i = \text{IngresoBase}\cdot\Big((1+\text{inflación})(1+\text{crec\_real})\Big)^i
\]

**Costos operativos ajustados**
\[
\text{Costos}_i = (C_f + C_v)\cdot f_{\text{infl}}(i)
\]
*(Nota: en el código, costos en Bs y en USD se suman sin convertir explícitamente la porción en Bs a USD; puede ajustarse para mayor rigor).*

**Tributos actividad económica**
\[
AE_i = 0.03\cdot \text{Ingreso}_i
\]

**ISLR mensual (aprox.)**  
Se calcula sobre la **ganancia neta en Bs** aplicando tramos y luego se convierte a USD.

**Flujo neto y acumulado**
\[
F_i = \text{Ingreso}_i - (\text{Costos}_i + AE_i + \text{Inversión}_i + \text{ISLR}_i)
\]
\[
FA_i = \sum_{k=0}^{i} F_k
\]

**Indicadores**
- **TIR**: `irr(flujos)`
- **VAN** (suma simple): \(\sum F_i\)
- **VPN**: `npv(WACC, flujos)`, con \(WACC=16\%\).

---

## Parámetros del Modelo

**Horizonte:**
- 15 meses (`Nov-2025` a `Ene-2027`).

**Ingresos y costos base (USD):**
- `ingreso_base = 15000`
- `costos_fijos_base = 5100`
- `costos_variables_base = 2040`

**Inversión (capex):**
- `gasto_inversion_total = 150000` en **3 meses** (`50000`/mes).

**Inflación anual por escenario:**
- `optimista: 10%`, `base: 7.5%`, `pesimista: 5%`.

**Crecimiento real ingresos:**
- `optimista: 10%`, `base: 7.5%`, `pesimista: 0.5%`.

**Tipo de cambio (Bs/USD):**
- `tc_inicial = 249.20`
- crecimiento anual: `optimista: 8%`, `base: 12%`, `pesimista: 15%`.

**Dolarización de costos:**
- `optimista: 65%`, `base: 70%`, `pesimista: 75%`.

**Tributos y fiscalidad:**
- Actividad Económica: `3%` sobre ingresos.
- **ISLR**: tramos (en Bs) aplicados a ganancia neta mensual.

**Descuento:**
- `WACC = 0.16` (16%) para **VPN**.

---

## Pseudocódigo

```bash
INPUT: meses, ingreso_base, costos_fijos, costos_variables,
       inversión_total (3 meses), inflación, crec_real,
       tc_inicial, tc_crecimiento, dolarización,
       tasa_actividad_económica, tramos_ISLR, WACC

FOR escenario in [optimista, base, pesimista]:
    flujo_acum = 0
    break_even = None
    FOR i, mes in enumerate(meses):
        f_infl = (1 + inflacion_esc)^i
        tc_mes = tc_inicial * (1 + tc_crec_esc)^i

        ingresos = ingreso_base * ((1+inflacion_esc)*(1+crec_real_esc))^i
        costos   = (costos_fijos + costos_variables) * f_infl
        tributos = 0.03 * ingresos
        inversion = (i < 3) ? inversion_total/3 : 0

        ganancia_neta = ingresos - (costos + tributos + inversion)
        islr = tramo_ISLR(ganancia_neta * tc_mes) / tc_mes   # si ganancia > 0

        salida_total = costos + tributos + inversion + islr
        flujo = ingresos - salida_total
        flujo_acum += flujo

        if (flujo_acum > 0 and break_even == None):
            break_even = mes

        registrar fila en DataFrame: (mes, tc_mes, ingresos, costos,
                                      tributos, inversion, islr,
                                      flujo, flujo_acum)

    TIR = irr(flujos)
    VAN = sum(flujos)
    VPN = npv(WACC, flujos)

    imprimir tabla, break_even, TIR, VAN, VPN

```

## Suposiciones y Alcances

* ISLR mensual: se estima por tramos con base en la ganancia del mes (aproximación; puede anualizarse).
* Dolarización: la porción en Bs puede convertirse a USD con tc_mes para mayor precisión contable.
* Ingresos: combinan efectos de inflación (precio) y crecimiento real (volumen) de forma multiplicativa.
* No se incluyen shocks de oferta/demanda, cambios regulatorios o riesgos operativos externos.


## Validación y Calidad de Resultados

* Coherencia de flujos: revisar que el flujo acumulado crezca conforme a supuestos.
* Punto de equilibrio: verificar el mes reportado con la tabla mensual.
* Benchmarks: comparar TIR/VPN contra tasas del sector y costo de capital real.
* Sensibilidades: stress-test de inflación, FX, crecimiento, WACC.



## Extensiones Sugeridas

* Rigor contable FX: separar costos en Bs → convertir a USD por tc_mes.
* Exportación: escribir hojas Excel (optimista/base/pesimista) + resumen KPIs; generar PDF.
* Gráficos: líneas de flujo mensual/acumulado, barras apiladas y marca de break-even.
* Sensibilidad: “tornado plots” para visualizar el impacto de supuestos.
* Calendario capex: permitir inversión no uniforme (por hitos).

