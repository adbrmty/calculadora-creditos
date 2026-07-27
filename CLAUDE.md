# CLAUDE.md — Notas para futuras sesiones de trabajo sobre este proyecto

Contexto operativo para quien retome `calculadora-compra.html`, sea persona o asistente.

---

## Arquitectura: un solo archivo, deliberadamente

`calculadora-compra.html` contiene HTML, CSS y JavaScript juntos. **No es descuido: es una decisión.** Se publica en GitHub Pages, la edita una sola persona y debe poder abrirse con doble clic sin servidor ni compilación.

**No hagas nada de esto sin autorización expresa del responsable del proyecto:**

- Migrar a React, Vue, Angular o TypeScript
- Separar CSS o JavaScript en archivos aparte
- Introducir npm, bundlers o cualquier paso de compilación
- Mover la lógica de cálculo a un backend
- Reescribir el archivo completo cuando bastan cambios localizados

La migración a Cloudflare Workers está en el horizonte, pero es una tarea futura con autorización propia.

---

## Reglas de cambio

1. **Inspecciona antes de modificar.** El archivo tiene interdependencias que no son obvias.
2. **Cambios mínimos y localizados.** Nada de refactorizaciones generales mientras corriges un error visual.
3. **No mezcles categorías.** Un cambio ortográfico y uno matemático no van en el mismo paso sin identificarlos por separado.
4. **No modifiques fórmulas financieras** salvo con evidencia matemática, un caso límite reproducido o instrucción explícita. Nunca las ajustes para hacer coincidir un número esperado.
5. **No toques identificadores al corregir ortografía.** Variables como `anios`, `anual`, `comision`, `avaluo`, `danos` son nombres de código. Solo se corrigen las **cadenas de texto visibles**. Un reemplazo global de "n" por "ñ" rompe el archivo.
6. **No cambies el endpoint de Formspree** (`mjgdeero`) sin autorización.
7. **No envíes leads ni mensajes de WhatsApp reales** en pruebas.

---

## Funciones críticas

Estas concentran el riesgo. Trátalas con cuidado.

| Función | Qué hace | Cuidado |
|---|---|---|
| `pagoMensual(P, tasaAnual, años)` | Anualidad estándar | Contempla tasa cero. Verificada contra `P·i/(1-(1+i)^-n)` con diferencia 0.00000000 |
| `amortizar(...)` | Recorre mes a mes | Devuelve totales y tabla anual. Los abonos se suman **tanto** a `accCap` como a `totCap`; omitir el segundo fue un bug real |
| `calcularCAT(...)` | TIR mensual anualizada por bisección | El NPV es **creciente** en *i* porque el flujo inicial es positivo. Invertir la dirección de la bisección produce cifras absurdas como 12874% |
| `num(id)` | Lee un campo y lo acota | Usa los `min`/`max` del propio HTML. Los campos `data-money` se leen solo como dígitos |
| `gastosEscritura(precio)` | Gastos de cierre | Notario lleva IVA (`×1.16`); la comisión se calcula sobre el **crédito**, no sobre el precio |
| `calcularInverso()` | Despeja el crédito máximo | El denominador combina anualidad y ambos seguros; el de daños se divide entre el LTV porque se calcula sobre el valor del inmueble, no sobre el crédito |

---

## Puntos donde es fácil equivocarse

**El comparador de bancos y el cálculo inverso viven fuera de los paneles.** Siempre se muestran, así que `recalcular()` los invoca en cualquier esquema. Si los mueves dentro de `calcularBancario()`, quedan congelados con datos viejos al cambiar a Infonavit o preventa. Ya ocurrió una vez.

**El desglose de Infonavit debe sumar exactamente el precio.** Los recursos se aplican en orden (subcuenta, crédito Infonavit, crédito bancario) con topes, precisamente para que las líneas cuadren cuando el crédito supera el valor de la propiedad.

**Cascada CSS de impresión.** `.print-only` tiene `display:none` en pantalla y `display:block` dentro de `@media print`. La regla `.print-head{display:flex}` debe ir **después** de `.print-only{display:block}` y **solo** dentro del bloque de impresión. Poner `display:flex` en la regla base hace que el encabezado de impresión aparezca en pantalla.

**Los campos de dinero son `type="text"`, no `type="number"`.** Los campos numéricos nativos no aceptan separadores de miles. Llevan `data-money`, `inputmode="numeric"` y se formatean al perder el foco. `num()` los limpia dejando solo dígitos.

**Los `input[type=range]` se auto-acotan; los de texto no.** Por eso `num()` aplica `min`/`max` manualmente.

---

## Interfaces que no deben romperse

- **IDs del DOM:** 75 identificadores referenciados desde JavaScript. Cambiar uno sin actualizar su contraparte rompe la función en silencio.
- **`BANCOS_REFERENCIA`:** campos `tipo`, `banco`, `producto`, `tasa`, `aforo`, `plazoMax`, `com`, `detalle`, `vig`, `vigISO`. El campo `com` está reservado: se conserva por producto pero hoy los cálculos usan el input global `#comision`.
- **`ASESORES`:** pares `[nombre, teléfono de 10 dígitos]`. El `52` se antepone al construir el enlace.
- **Clases con significado funcional:** `adv-only` (solo modo avanzado), `no-print`, `print-only`, `data-money`.
- **Nombres de campo de Formspree:** `nombre`, `telefono`, `email`, `escenario`, `asesor`, `consentimiento`, `_subject`, `_gotcha`.

---

## Comandos de validación

Requiere `npm install jsdom`.

```bash
# Sintaxis e integridad de IDs
node -e "
const fs=require('fs');const html=fs.readFileSync('calculadora-compra.html','utf8');
const js=html.match(/<script>([\s\S]*?)<\/script>/)[1];
new Function(js); console.log('Sintaxis OK');
const ids=[...new Set([...js.matchAll(/\\\$\('([^']+)'\)/g)].map(x=>x[1]))];
const falta=ids.filter(i=>!html.includes('id=\"'+i+'\"'));
console.log('IDs faltantes:', falta.length?falta.join(', '):'ninguno');
"

# Suite completa (79 aserciones)
node test_final.js

# Verificación financiera independiente
node verif2.js
```

La suite cubre carga, identidad, formato de dinero, ortografía, CAT, vigencias, sincronía del comparador, cuadre de Infonavit, selector de asesor, privacidad, impresión, regresiones y modo simple.

**Al corregir ortografía, excluye `<script>` y `<style>` antes de buscar.** Si no, los nombres de variables aparecen como falsos positivos. Ocurrió: `comision` y `Avaluo` se reportaron como errores siendo identificadores legítimos.

---

## Lo que las pruebas automáticas no cubren

jsdom no calcula layout ni imprime. Estas verificaciones exigen un navegador real:

- Renderizado y desbordamiento en 360 / 390 / 768 / 1280 / 1920 px
- Vista previa de impresión en tamaño Carta: saltos de página, tablas partidas, gráficos de fondo
- Gráficas de Chart.js
- Entrega efectiva de Formspree
- Apertura real de WhatsApp
- Funcionamiento sobre GitHub Pages

---

## Cifras de referencia

Escenario por defecto: propiedad $3,500,000, enganche 20%, plazo 20 años, tasa 9.5%, seguros 0.55% y 0.35%, comisión 1%.

| Concepto | Valor |
|---|---|
| Crédito | $2,800,000 |
| Capital e intereses | $26,100 |
| Mensualidad con seguros | $28,404 |
| Gastos de escrituración | $213,400 (6.1%) |
| Efectivo para iniciar | $913,400 |
| CAT estimado | 11.4% |

Si tras un cambio estas cifras se mueven sin que lo hayas buscado, hay una regresión.
