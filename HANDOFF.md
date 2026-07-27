# HANDOFF — Calculadora de compra de propiedad ADBR

**Fecha del documento:** 25 de julio de 2026
**Archivo:** `calculadora-compra.html` (~87 KB, archivo único)
**Estado:** candidato para publicación, con riesgos declarados

---

## 1. Estado actual

La herramienta pasó de un prototipo funcional pero con defectos de presentación y coherencia, a una versión revisada, probada y con identidad de marca.

**Suite automatizada: 79 aserciones, 0 fallos, 0 errores de ejecución.**

Se corrigieron 2 hallazgos críticos, 5 altos, 9 medios y 11 bajos, además de 2 defectos introducidos durante la propia corrección y detectados en la auditoría adversarial.

---

## 2. Decisiones tomadas

| Tema | Decisión |
|---|---|
| Logotipos | Tres archivos oficiales entregados. Embebidos en base64 (horizontal blanco en encabezado, horizontal a color en impresión, cuadro como favicon) |
| Aviso de privacidad | `https://www.adbr.mx/privacy`, con casilla de aceptación obligatoria |
| Créditos SOC | Solo mención textual. Sin logotipo, a falta de autorización revisada para uso de marca |
| Infonavit / Cofinavit | Se mantiene como estimador simplificado; solo se corrigió el descuadre |
| CAT | Incorpora el avalúo, con explicación breve en la interfaz |
| Comparador fuera del esquema bancario | Se recalcula siempre desde el escenario bancario, con nota que lo aclara |
| Modo por defecto | Simple |
| Requisitos de calificación | Visibles también en modo simple |
| Campos editables en modo simple | No: seguros y gastos permanecen solo en avanzado |
| WhatsApp | Oficina 81 8143 1003, más selector de asesor con contacto directo |
| Formato monetario | Pesos enteros, sin decimales; precisión completa en el cálculo interno |
| PDF | Incluye logotipo, fecha, esquema y parámetros; sin datos personales |

**Supuesto adoptado:** para el destino de publicación se confirmó "la misma" ubicación. El archivo no contiene rutas absolutas, por lo que funciona desde cualquier repositorio o subruta sin ajustes.

---

## 3. Correcciones aplicadas

### Confiabilidad

- **Entradas fuera de rango.** `num()` acota con los `min`/`max` declarados en el HTML. Antes, un precio negativo producía una mensualidad de **−$4,058** y un desembolso de **−$120,200**.
- **Descuadre en Infonavit.** Los recursos se aplican con topes en orden. Antes, con precio $1,000,000, crédito $900,000 y subcuenta $180,000, las líneas sumaban $1,080,000 contra un total de $1,000,000. Ahora cuadra en los cuatro casos probados, y se avisa del crédito no utilizado.
- **Comparador congelado.** `renderBancos()` pasó de `calcularBancario()` a `recalcular()`. Antes mostraba cifras del esquema anterior al cambiar de panel.
- **Bug latente en abonos a capital.** Los abonos se sumaban al acumulador anual pero no al global: `totCap` daba $2,100,000 en lugar de $2,800,000. No se manifestaba porque solo se usaba con abono cero, pero habría aparecido al reutilizar la función.
- **Etiqueta financiera incorrecta.** La gráfica de dona rotulaba el monto del crédito como "valor de la casa".
- **Texto de aforo desactualizado.** Decía 80% típico cuando los 13 productos declaran 90%.

### Información financiera

- **Nueve campos de dinero** migrados a `type="text"` con `inputmode="numeric"` y formateo con separadores. Se editan en crudo al enfocar y se reformatean al salir. Lo mostrado siempre coincide con lo calculado, incluso al aplicar topes.
- **CAT con avalúo:** 11.30% → 11.35%, más cerca del 11.5% que reporta Créditos SOC. Control de validación: sin seguros, comisión ni avalúo devuelve 9.92%, que es la tasa efectiva de un 9.5% nominal.
- **Vigencias verificables:** cada producto lleva `vigISO`. Los vencidos se marcan solos; los que no tienen fecha declaran "vigencia por confirmar" en lugar de un guion.
- **Mensaje de WhatsApp** con esquema declarado, advertencia de estimación y ortografía corregida.

### Presentación

- **Ortografía:** 16 correcciones en texto visible, incluidas seis apariciones de "ano"/"anos" en las etiquetas de la gráfica, el comparador de plazos y el botón de la tabla. Los identificadores no se tocaron.
- **Identidad:** logotipo en encabezado, favicon, Open Graph.
- **Leyenda institucional** de ADBR y Créditos SOC, con aclaración de que ADBR no es institución financiera.
- **Bloque de impresión** con logotipo, fecha y resumen de parámetros. Antes existía la regla CSS pero ningún elemento la usaba.
- **Selector de asesor** con los 23 asesores; el mensaje saluda por nombre y va a su teléfono.
- **Privacidad:** casilla obligatoria enlazada al aviso, más trampa antispam.
- **Accesibilidad:** dos `<label>` decorativos convertidos en `<span>`.
- **Rendimiento:** escribir en el formulario de contacto ya no recalcula toda la proyección.

### Defectos introducidos durante la corrección y resueltos

1. `.print-head` tenía `display:flex` en la regla base, lo que hacía **visible en pantalla** el encabezado de impresión. Verificado con `getComputedStyle`: ahora devuelve `none`.
2. Una regla `.print-only` duplicada quedaba después de `.print-head`, anulando su layout flex al imprimir.

---

## 4. Validación financiera

| Prueba | Resultado |
|---|---|
| Anualidad contra fórmula independiente (4 escenarios) | Diferencia 0.00000000 |
| Amortización: cierre de saldo (3 escenarios) | $0.00, meses exactos |
| Capital acumulado contra principal | Diferencia 0.00 |
| Abonos: $50,000 anuales | 20 → 14.2 años, ahorro $1,141,721 |
| CAT control sin cargos | 9.92% (tasa efectiva de 9.5% nominal) |
| CAT con supuestos por defecto | 11.35% frente a 11.5% de SOC |

**Escenario de referencia:** propiedad $3,500,000, enganche 20%, 20 años, 9.5% → crédito $2,800,000, capital e intereses $26,100, mensualidad con seguros $28,404, gastos $213,400 (6.1%), efectivo inicial $913,400, CAT 11.4%.

---

## 5. Productos vigentes

13 productos: 9 de vivienda, 4 de terreno. Fuente: Créditos SOC, corte julio 2026.

Tasas de vivienda entre **8.80%** (Banorte, tres campañas) y **10.75%** (Scotiabank Vinculación). Aforo hasta 90%. Plazo hasta 20 años sin penalización por prepago.

**Atención inmediata:** dos productos vencen el **31 de julio de 2026**, dentro de días:

- Mifel · Promoción Mundialista (9.60%)
- Mifel · Lanzamiento Economía Americana (9.90%)

A partir del 1 de agosto la herramienta los marcará automáticamente como *"VIGENCIA VENCIDA, confirmar condiciones"*. No requiere intervención urgente, pero conviene pedir a Marco las condiciones de agosto antes de esa fecha.

Tres productos no tienen vigencia declarada y muestran *"vigencia por confirmar"*: BX+, Scotiabank Cliente Affluent y Scotiabank Vinculación. Vale la pena solicitarlas.

---

## 6. Riesgos residuales

| # | Riesgo | Severidad | Mitigación aplicada | Acción pendiente |
|---|---|---|---|---|
| 1 | Sin verificación en navegador real: layout, responsive, impresión y gráficas | **Alto** | Revisión de CSS, breakpoints y reglas de impresión | Abrir en Chrome, revisar en cinco anchos y hacer vista previa de impresión |
| 2 | Formspree no verificado de extremo a extremo | Medio | Estructura, endpoint y manejo de errores revisados | Un envío de prueba controlado |
| 3 | Dos productos vencen el 31 de julio | Medio | Marcado automático al vencer | Pedir condiciones de agosto |
| 4 | Tres productos sin vigencia declarada | Bajo | Se declara "por confirmar" | Solicitar fechas a SOC |
| 5 | Uso de marca de Créditos SOC no formalizado | Medio | Solo mención textual, sin logotipo | Confirmar por escrito la redacción |
| 6 | El CAT no coincide exactamente con el oficial | Bajo | Declarado como estimado en interfaz y aviso legal | Ninguna |
| 7 | Infonavit sigue siendo estimador simplificado | Bajo | Nota explícita en el panel | Decidir si se profundiza |
| 8 | Lógica de cálculo visible en el navegador | Bajo | Inherente a una página estática | Migración a Cloudflare Workers |
| 9 | Chart.js depende de cdnjs | Bajo | La herramienta funciona sin gráficas si falla | Considerar alojarlo |
| 10 | Sin repositorio Git | Bajo | Respaldo del original conservado | Inicializar control de versiones |

---

## 7. Recorrido de demostración para el asesor

Cinco minutos, en este orden:

1. **Abre en modo simple.** "Esta es la propiedad que te interesa." Ajusta el precio.
2. **Mueve el enganche.** El cliente ve cómo cambia la mensualidad en tiempo real.
3. **Baja al desembolso inicial.** Aquí está el golpe de realidad: *"además del enganche de $700,000, escriturar cuesta $213,400. Necesitas $913,400 para arrancar."* Este es el momento que justifica la herramienta.
4. **Cambia a modo avanzado.** Muestra el comparador de plazos: a 20 años la mensualidad es cómoda, pero los intereses totales duplican el crédito.
5. **Comparador de bancos.** *"Con tu perfil podrías calificar a 8.80% en lugar de 9.5%; son $1,266 menos al mes."*
6. **Simulador de abonos.** *"Si abonas tu aguinaldo, terminas en 14 años en vez de 20 y ahorras $1.1 millones."*
7. **Cierra.** Selecciona tu nombre en "Ya tengo asesor" e imprime el PDF, o envía la proyección por WhatsApp.

---

## 8. Tareas futuras

1. Verificación en navegador (bloquea la publicación definitiva)
2. Condiciones de agosto de Créditos SOC
3. Vigencias faltantes de tres productos
4. Formalizar la redacción sobre la colaboración con SOC
5. Inicializar Git
6. Evaluar si Infonavit merece modelo completo
7. Migración a Cloudflare Workers (requiere autorización: rompe la arquitectura de archivo único)

---

## 9. Veredicto

**LISTO PARA PUBLICACIÓN CON RIESGOS ACEPTADOS.**

Todo lo verificable en este entorno está verificado y en verde. El único riesgo alto es que ninguna herramienta disponible aquí renderiza páginas: layout, responsive, impresión y gráficas se revisaron leyendo el CSS, no viéndolo funcionar.

La recomendación es abrir el archivo en Chrome y recorrer la lista del punto 6.1 antes de publicar. Si el recorrido sale limpio, la herramienta puede subirse sin reservas.
