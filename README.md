# Calculadora de compra de propiedad — AD Bienes Raíces Monterrey

Herramienta web que proyecta la compra de una propiedad con crédito bancario, Infonavit/Cofinavit o preventa. Está pensada para dos usos simultáneos: consulta pública desde el sitio de ADBR y apoyo del asesor durante una cita.

Desarrollada en colaboración con **Créditos SOC**, que aporta las condiciones vigentes de los productos hipotecarios.

---

## Qué la diferencia

La mayoría de los simuladores muestran únicamente capital e intereses. Esta calculadora responde las dos preguntas que realmente decide el comprador:

1. **Cuánto necesito el día de la compra.** Enganche más ISAI, honorarios notariales con IVA, derechos de registro, avalúo y comisión de apertura. En el escenario por defecto son **$913,400** sobre una propiedad de **$3,500,000**, no los $700,000 del enganche.
2. **Cuánto pago realmente al mes.** La cifra protagonista incluye los seguros de vida y daños, que van dentro de la mensualidad del banco. Son **$28,404**, no los $26,100 de capital e intereses.

---

## Funcionalidades

**Esquemas de compra**
- Crédito bancario (completo)
- Infonavit / Cofinavit (estimador simplificado)
- Preventa con desarrollador

**Dos niveles de detalle**
- Simple: vista por defecto, orientada al cliente
- Avanzado: añade seguros y gastos editables, CAT, comparadores, amortización y abonos

**Herramientas**
- Desglose del desembolso inicial
- Composición del pago mensual
- CAT estimado
- Comparador de plazos con gráficas
- Tabla de amortización anual
- Simulador de abonos a capital
- Comparador de 13 productos bancarios con filtro vivienda/terreno
- Cálculo inverso de capacidad de compra
- Requisitos de calificación

**Contacto**
- Selector de asesor: quien ya es atendido por un asesor de ADBR contacta directamente con él; el resto llega a la oficina
- Formulario con Formspree y consentimiento de privacidad
- Exportación a PDF desde el navegador

---

## Estructura

Un solo archivo: `calculadora-compra.html` (~87 KB), con HTML, CSS y JavaScript juntos.

| Sección | Contenido |
|---|---|
| `<head>` | Metadatos, Open Graph, favicon embebido, tipografías, Chart.js |
| `<style>` | Variables de marca, componentes, responsive, reglas de impresión |
| `<body>` | Encabezado, controles, tres paneles, herramientas, CTA, aviso legal |
| `<script>` | Catálogo de productos, asesores, cálculo, render, eventos |

**Sin dependencias de compilación.** Se abre directamente en el navegador.

Recursos externos: Google Fonts y Chart.js 4.4.1 desde cdnjs. Los logotipos y el favicon van embebidos en base64, de modo que no hay rutas que se puedan romper.

---

## Cómo abrirla localmente

Basta con abrir `calculadora-compra.html` con doble clic. Si prefieres servirla:

```bash
python3 -m http.server 8000
# luego abre http://localhost:8000/calculadora-compra.html
```

---

## Cómo publicar en GitHub Pages

1. Copia `calculadora-compra.html` a la raíz del repositorio.
2. Haz commit y push a la rama que sirve Pages (`main` o `gh-pages`).
3. En **Settings → Pages**, confirma la rama y la carpeta `/ (root)`.
4. La URL queda como `https://<usuario>.github.io/<repositorio>/calculadora-compra.html`.

Funciona igual desde una subruta: no hay rutas absolutas al dominio ni referencias locales.

**Después de publicar, verifica:** que carguen las tipografías y las gráficas, que el logotipo se vea en el encabezado, que el botón de WhatsApp abra la conversación y que la vista previa de impresión muestre el encabezado con logo y fecha.

---

## Cómo actualizar los productos bancarios

Créditos SOC publica condiciones nuevas cada mes. La actualización se hace en **un solo lugar**.

Abre `calculadora-compra.html`, busca `const BANCOS_REFERENCIA` (al inicio del `<script>`) y edita la lista.

```js
{ tipo:"vivienda",              // "vivienda" o "terreno"
  banco:"Banorte",
  producto:"Nominahabientes",
  tasa:8.80,                    // número, o null si es "a cotizar"
  aforo:90,                     // % máximo de financiamiento
  plazoMax:20,                  // años; el comparador ajusta si el usuario elige más
  com:0,                        // % de comisión de apertura (informativo)
  detalle:"Texto de condiciones",
  vig:"31 dic 2026",            // lo que se muestra
  vigISO:"2026-12-31" },        // lo que se compara con la fecha de hoy
```

**`vig` y `vigISO` deben corresponder.** `vigISO` permite que la herramienta marque sola los productos vencidos: cuando la fecha pasa, la fila muestra *"VIGENCIA VENCIDA, confirmar condiciones"* en lugar de la fecha. Si un producto no tiene vigencia definida, deja ambos campos vacíos (`""`) y aparecerá *"vigencia por confirmar"*.

Después actualiza la constante de la línea siguiente:

```js
const FECHA_ACTUALIZACION = "julio 2026";
```

Esa constante alimenta el chip del encabezado, la insignia del comparador y el pie del PDF, así que no hay que tocarlos por separado.

Si cambia el rango de tasas, ajusta también el texto del encabezado (`id="rate-range"`) y el atajo `mercado 8.80% – 10.75%` de la etiqueta de tasa.

**Validación posterior:** abre la página, cambia al modo avanzado, revisa el comparador en vivienda y terreno, y confirma que ningún producto muestre *"VIGENCIA VENCIDA"* sin que sea intencional.

---

## Cómo actualizar la lista de asesores

Busca `const ASESORES` y edita la lista de pares `["Nombre", "teléfono a 10 dígitos"]`:

```js
const ASESORES = [
  ["Abil Razo","8122038273"],
  ...
];
```

El prefijo `52` se agrega automáticamente al construir el enlace de WhatsApp. La lista aparece ordenada tal como se escribe, así que conviene mantenerla alfabética.

El número de la oficina está en `const WA_OFICINA = "528181431003"` (con el 52 incluido).

---

## Cómo cambiar el formulario de contacto

El endpoint está en el atributo `action` del formulario:

```html
<form id="leadForm" action="https://formspree.io/f/mjgdeero" method="POST">
```

Campos enviados: `nombre`, `telefono`, `email`, `escenario` (oculto, con el esquema y las cifras), `asesor` (oculto), `consentimiento` y `_subject`. El campo `_gotcha` es una trampa antispam invisible; Formspree descarta los envíos que lo rellenen.

El aviso de privacidad apunta a `https://www.adbr.mx/privacy`.

---

## Cómo reemplazar los logotipos

Los logotipos están embebidos como `data:image/png;base64,...` en tres lugares: el `<link rel="icon">`, el `<img class="brand-logo">` del encabezado y el `<img class="print-logo">` del bloque de impresión.

Para sustituirlos:

```bash
base64 -w0 nuevo-logo.png
```

y reemplaza la cadena posterior a `base64,`. Conviene redimensionar antes (el horizontal a unos 460 px de ancho, el favicon a 64×64) para no inflar el archivo.

Alternativamente puedes usar archivos externos con rutas relativas (`src="logo.png"`), pero entonces habrá que subirlos junto al HTML.

---

## Limitaciones conocidas

- El CAT es **estimado**, no el CAT oficial de ninguna institución. Incluye tasa, seguros, comisión de apertura y avalúo.
- Infonavit/Cofinavit es un **estimador simplificado**: no modela enganche en efectivo ni gastos de escrituración, y usa la tasa que se capture.
- Los gastos de escrituración son porcentajes de referencia para Nuevo León y varían por notaría y por el valor sobre el que se calcule el ISAI.
- La lógica de cálculo es visible en el navegador, como en cualquier página estática.
- Las gráficas dependen de Chart.js desde CDN; si no carga, el resto de la herramienta sigue funcionando.

---

## Siguiente etapa

Migrar la lógica de cálculo a **Cloudflare Workers** para proteger las fórmulas y permitir repositorio privado. No está implementado y no debe abordarse sin autorización expresa: implica dividir la arquitectura de archivo único.
