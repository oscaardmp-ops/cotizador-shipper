# 📦 Cotizador · Shipper Perú

> Herramienta web para cotizar importaciones desde EE.UU. a Perú en régimen courier (hasta US$ 2,000).  
> Archivo único `index.html` — sin dependencias, sin backend, sin instalación.

---

## 🚀 Cómo desplegarlo en GitHub Pages

1. Crear un repositorio nuevo en GitHub (ej. `cotizador-shipper`)
2. Subir el archivo renombrado como `index.html`
3. Ir a **Settings → Pages → Branch: main → / (root)** → Save
4. En 1–3 minutos estará disponible en `https://tu-usuario.github.io/cotizador-shipper`

**Para actualizar:** reemplazar el `index.html` directamente desde el repo (botón lápiz o "Upload files").

---

## ✨ Funcionalidades

### Cotización básica
- Ingreso de **valor del producto** (US$) y **peso** (kg)
- Cálculo automático de:
  - **Envío** por peso: $10 / kg
  - **Desaduanaje** por tramo de valor
  - **Impuestos SUNAT** (Ad valorem 4% + IGV 16% + IPM 2%) solo si el valor supera US$ 200
- Conversión a **soles** con tipo de cambio actualizado (se intenta obtener el valor del día; si falla usa S/ 3.50 como referencial)
- Todos los montos en soles son **referenciales**

### Detección inteligente de peso
Al escribir la descripción del producto, el sistema detecta automáticamente la categoría y sugiere el peso promedio. Reconoce más de **100 palabras clave** en español e inglés:

| Categoría | Ejemplos detectados | Peso sugerido |
|---|---|---|
| Ropa | polo, hoodie, pants, shirt, jacket | 1.0 kg |
| Cosméticos | skincare, serum, makeup | 0.6 kg |
| Zapatillas | nike, adidas, jordan, sneakers | 1.4 kg |
| Celular | iphone, samsung, pixel, galaxy | 0.4 kg |
| Suplementos | whey, protein, vitamins | 0.8 kg |
| Laptop | macbook, laptop, notebook, lenovo | 2.7 kg |
| Audífonos | airpods, headphones, beats, bose | 0.4 kg |
| Mochila | backpack, bag, mochila | 1.4 kg |
| Consola | ps5, xbox, switch, nintendo | 1.0 kg |
| Smartwatch | apple watch, galaxy watch, fitbit | 0.4 kg |
| Monitor | monitor, display, pantalla | 2.3 kg |
| Tablet | ipad, tablet | 1.1 kg |
| Accesorios Auto | brake, filter, repuesto | 1.9 kg |
| Cámara | canon, gopro, nikon, fujifilm | 0.5 kg |

> La detección usa **word boundaries** (regex `\b`) para evitar falsos positivos. Ej: "laptop" no matchea con "top" (ropa).

Si no se detecta automáticamente, hay un estimador desplegable "¿No sabes el peso?" con las categorías más comunes.

---

## 📐 Lógica de desaduanaje

| Valor del producto | Tarifa |
|---|---|
| $0 – $50 | $4.00 |
| $50.01 – $100 | $7.00 |
| $100.01 – $200 | $11.00 |
| $200.01 – $2,000 | $11.00 + SUNAT |

### Múltiples compras
- Productos **≤ $200 c/u**: se suman y pagan **un solo desaduanaje** basado en la suma total
- Productos **> $200 c/u**: se evalúan de forma **independiente** y generan impuestos SUNAT por separado
- El peso siempre es el total de todos los productos

**Ejemplos:**

| Productos | Desaduanaje |
|---|---|
| $150 + $40 = $190 | 1× $11 (tramo $100–$200) |
| $150 + $70 = $220 | 1× $11 (tramo $100–$200) |
| $250 + $40 | $250 → $11 + SUNAT / $40 → $4 |

---

## 🎟️ Códigos de descuento

Los descuentos se aplican ingresando un código en la sección "¿Tienes un descuento?". Solo afectan las tarifas de Shipper, **nunca los impuestos SUNAT**.

| Código | Comunidad | Descuento |
|---|---|---|
| `SHP-NEW10-RV` | 👶 Shippitos (primera compra) | 10% envío + 10% desaduanaje |
| `SHP-FRQ10-MZ` | 💙 ShipperLovers (clientes frecuentes) | 10% envío + 10% desaduanaje |
| `SHP-FAM15-KQ` | 💛 Los Shippis F&F (amigos y familia) | 15% envío + 15% desaduanaje |
| `SHP-PRV1-XW8` | 💎 ShippiVIPS (proveedores nivel 1) | 20% envío + 20% desaduanaje |
| `SHP-PRV2-NJ4` | 😎 ShippiBOSSES (proveedores nivel 2) | 25% envío + 25% desaduanaje |
| `SHP-CRT5-HB` | 💚 Shippibos (cortesía) | 5% envío + 5% desaduanaje |

### Código variable ⚡ ShippiGODS (proveedores nivel 3)
Formato: `{PP}SHIPR{DD}` donde `PP` = tarifa de envío × 10 y `DD` = tarifa de desaduanaje × 10.

```
Ejemplo: 75SHIPR75
  PP = 75 → $7.50 por kg de envío
  DD = 75 → $7.50 tarifa de desaduanaje
```

Rango válido: 1–20 para cada campo (tarifas entre $0.10 y $2.00 × 10 = $0.10–$20.00).

```
Ejemplos de códigos ShippiGODS:
  75SHIPR75  → $7.50 / kg · $7.50 desaduanaje
  80SHIPR60  → $8.00 / kg · $6.00 desaduanaje
  50SHIPR50  → $5.00 / kg · $5.00 desaduanaje
```

> ⚠️ Los códigos son sensibles a mayúsculas internamente pero el campo los convierte automáticamente. Compártelos solo de forma directa y personal.

---

## 🛒 Servicio Concierge

Compramos el producto en EE.UU. por el cliente. Estructura de cobro:

```
Pago adelantado = Valor del producto + $1 fijo + 1% del valor
```

Ejemplo: producto de $200
```
  Valor del producto:  $200.00
  Comisión ($1 + 1%):   $3.00
  ─────────────────────────────
  Total a adelantar:  $203.00
```

> El cliente paga este monto **antes** de que se realice la compra. El cotizador lo indica explícitamente en el desglose y en el mensaje de WhatsApp.

---

## 🚚 Opciones de entrega en Perú

| Opción | Cobertura | Costo |
|---|---|---|
| 🏬 Recojo en almacén Shipper | Lima | Gratis |
| 🚚 Shalom | Todo Perú | Sin costo adicional (contraentrega) |
| 📦 Olva Courier | Todo Perú | Desde $4 |
| ⚡ Delivery express | Solo Lima (según cobertura) | Desde $4 |

> Shipper no cobra por trasladar el paquete hasta la agencia de Olva o Shalom.

---

## 📲 Mensaje de WhatsApp

Al completar la cotización se genera automáticamente un mensaje formateado para WhatsApp con:

- Descripción y nombre del cliente
- Desglose de productos (modo múltiple)
- Detalle de cobros Shipper con precios originales y con descuento
- 🔥 Ahorro total cuando hay código aplicado
- Impuestos SUNAT aproximados (si aplica)
- Total estimado en USD y soles
- Nota de concierge con pago adelantado (si aplica)
- Nota de Price Match Warranty

El botón **"Enviar por WhatsApp"** abre `wa.me/51989456429` con el mensaje prellenado. El cliente puede copiarlo también con el botón **"Copiar"**.

---

## 💾 Historial

- Las cotizaciones se **guardan automáticamente** 1.8 segundos después del último cambio
- Se almacenan en `localStorage` del navegador (hasta 30 entradas)
- Se pueden **restaurar** con un clic (recarga todos los campos del formulario)
- Se pueden eliminar individualmente o limpiar todas
- Al restaurar en modo múltiple, se recrean todos los productos correctamente

> El historial es local al dispositivo/navegador. No se sincroniza entre dispositivos.

---

## 🤝 Price Match Warranty

Si un cliente presenta una cotización más barata de otro courier, Shipper puede igualar el precio. Se menciona en:
- Los servicios adicionales (con tooltip explicativo)
- Al pie de cada mensaje de WhatsApp generado

---

## ✈️ Servicio Viajero

Aparece automáticamente en la cotización **solo cuando hay productos que pagan SUNAT** (valor individual > $200). El botón de WhatsApp incluye el nombre del producto y su valor en el mensaje prellenado para consulta directa.

---

## 🧰 Otros servicios (informativos)

No se incluyen en la cotización. Son solo referencia:

| Servicio | Precio |
|---|---|
| Foto de contenido | Desde $5 |
| Consolidación | Gratis hasta 5 paquetes |
| Separación | Desde $5 / paquete |
| Pickup en EE.UU. | Desde $15 |
| In/Out (sin envío a Perú) | Desde $25 |
| Bubble wrap | Desde $5 |

---

## 🏗️ Arquitectura técnica

```
cotizador/
└── index.html          # Archivo único — todo en uno
    ├── <style>         # CSS con variables de brand Shipper (~ 200 líneas)
    ├── <html>          # Formulario + resultado + historial + servicios
    └── <script>        # Lógica JS en modo estricto (~ 500 líneas)
        ├── CONFIG      # Constantes globales (frozen object)
        ├── DCODES      # Códigos de descuento
        ├── SVCS        # Servicios adicionales
        ├── PESOS       # Tabla de pesos por categoría con keywords
        ├── STATE       # Estado global del módulo
        ├── fetchTC()   # Tipo de cambio desde APIs externas
        ├── detectarPeso()   # Detección por word boundaries
        ├── validate()       # Validación de inputs
        ├── calcular()       # Motor de cálculo principal
        ├── renderResultado()# Genera HTML del resultado
        ├── buildMsg()       # Genera mensaje de WhatsApp
        ├── scheduleAutoSave() # Auto-guardado con debounce
        └── renderHist()     # Historial desde localStorage
```

### Dependencias externas
- **Google Fonts** — Montserrat + JetBrains Mono (solo tipografía)
- **exchangerate-api.com** + **open.er-api.com** — tipo de cambio (con fallback a S/ 3.50)

Sin frameworks, sin npm, sin build step.

---

## 🎨 Brand & diseño

Paleta oficial Shipper Perú:

| Token | Hex | Uso |
|---|---|---|
| `--brand` | `#7ebdc2` | Color principal (teal) |
| `--slate` | `#394f5b` | Fondo oscuro, headers |
| `--ink` | `#1f1d22` | Texto principal |
| `--gold` | `#f3bb53` | Acento, impuestos SUNAT |

Tipografía: **Montserrat** (cuerpo) — tipografía recomendada del brand kit oficial.

---

## ⚠️ Limitaciones y notas

- Régimen courier SUNAT: máximo **US$ 2,000** por envío. Sobre ese valor se requiere agente de aduanas.
- Los impuestos SUNAT son **referenciales**. El monto exacto lo determina SUNAT al momento del despacho.
- El tipo de cambio es siempre **referencial**, independientemente de si se obtuvo en tiempo real.
- Los tiempos de entrega son **2 a 5 días hábiles** en promedio desde EE.UU.
- El historial se pierde si el usuario limpia los datos del navegador.

---

## 🔧 Personalización rápida

### Cambiar tarifas base
En `<script>`, sección `CFG`:
```js
PESO_RATE: 10,      // $10 por kg — cambiar aquí
CONCIERGE_FIJO: 1,  // $1 fijo concierge
CONCIERGE_PCT: 0.01 // 1% del valor
```

### Agregar/cambiar códigos de descuento
En `DCODES`:
```js
'NUEVO-CODIGO': { type:'pct', dp:20, da:20, label:'🌟 Nombre', desc:'Descripción...' }
```

### Cambiar número de WhatsApp
En `CFG`:
```js
WA: '51989456429'  // Formato: código país + número, sin +
```

### Agregar categorías al estimador de peso
En `PESOS`:
```js
{e:'🖨️', n:'Impresora', p:4.5, keys:['printer','impresora','hp printer','epson','canon printer']}
```

---

*Desarrollado para Shipper Perú · @shipper.pe*
