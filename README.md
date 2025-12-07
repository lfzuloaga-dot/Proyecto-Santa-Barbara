
# Evaluación Financiera del Taller Santa Bárbara

## Propósito

Modelar la viabilidad financiera del taller **Santa Bárbara** mediante simulación de escenarios (optimista, base y pesimista). Se proyectan ingresos, costos, tributos, inversión inicial y flujos mensuales; además, se calculan **TIR**, **VAN**, **VPN (WACC)** y el **punto de equilibrio**.

## Índice

* Visión General
* Fundamentos Matemáticos
* Parámetros del Modelo
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


## Fundamentos Matemáticos

**Actualización mensual de índices**

<img width="356" height="28" alt="image" src="https://github.com/user-attachments/assets/494c2963-198d-4809-bc86-11ca9004a45a" />


**Ingresos proyectados**

<img width="359" height="26" alt="image" src="https://github.com/user-attachments/assets/64734097-bbc6-420e-a117-17a09c8616d3" />

**Costos operativos ajustados**

<img width="164" height="28" alt="image" src="https://github.com/user-attachments/assets/aca04f9a-de0e-48c3-97dd-16114d84fb20" />

**Tributos actividad económica**

<img width="253" height="29" alt="image" src="https://github.com/user-attachments/assets/50ee0cee-11e0-4a46-b608-ed7e7d5d9494" />



**Flujo neto**

<img width="418" height="44" alt="image" src="https://github.com/user-attachments/assets/57d72ae0-c071-45d5-a181-13ac5d86fa08" />


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

