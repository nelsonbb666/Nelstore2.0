# NELSTORE — Sprint 2 · Esquema ACTUALIZADO de `data/products.json`

> Versión 2.0 del esquema (reemplaza y amplía `docs/05-esquema-products-json.md`).
> Nuevos campos exigidos por el sprint: **`variantes`, `video`, `registroSanitario`**
> (renombrado desde `invima`), más reglas para que el mensaje de WhatsApp salga siempre completo.

## 1. Producto completo (comentado)

```jsonc
{
  "meta": { "version": "2.0", "moneda": "COP", "actualizado": "2026-10-09" },
  "productos": [
    {
      // ── IDENTIFICACIÓN ─────────────────────────────────────────────
      "id": "p001",                             // estable, clave interna de la plantilla
      "sku": "NEL-MAS-001",                     // ✔ NUEVO-CONFIRMADO: referencia visible; viaja en el mensaje WA ("Referencia:")
      "slug": "pistola-de-masaje-pro-x",        // URL /producto/{slug}
      "nombre": "Pistola de Masaje Pro X 6 Velocidades",

      // ── CLASIFICACIÓN ──────────────────────────────────────────────
      "categoria": "masajeadores",              // slug existente en categories.json
      "etiquetas": ["masajeador", "pistola", "recuperacion", "muscular"],  // buscables, sin tildes

      // ── PRECIO ─────────────────────────────────────────────────────
      "precio": 129900,                         // entero COP → "$129.900" con toLocaleString('es-CO')
      "precio_anterior": 179900,                // null si no hay descuento; % se calcula al vuelo
      "oferta": true,
      "oferta_hasta": "2026-10-15",             // null = permanente; alimenta contador de /ofertas

      // ── IMÁGENES Y VIDEO ───────────────────────────────────────────
      "imagenes": [
        "assets/img/productos/p001-1.webp",     // [0] principal (card + og:image)
        "assets/img/productos/p001-2.webp",
        "assets/img/productos/p001-3.webp"
      ],
      "imagen_alt": "Pistola de masaje negra con 4 cabezales",
      "video": {                                // ✔ NUEVO — OPCIONAL (null u omitido = no se pinta el bloque)
        "mp4": "assets/video/p001-uso.mp4",     // ≤15 s, vertical o cuadrada, <2 MB; NO se descarga hasta pulsar play
        "poster": "assets/img/productos/p001-video-poster.webp",
        "duracion_s": 14,
        "alt": "Pistola de masaje en uso sobre el hombro"
      },

      // ── VARIANTES ──────────────────────────────────────────────────
      // ✔ NUEVO — grupos arbitrarios (color/talla/presentación). Cada variante puede
      // sobrescribir precio/sku/imagen. La elegida se agrega al mensaje WA:
      // "• Variante (Color): Negro mate". Sin variantes → fieldset no se renderiza.
      "variantes": [
        {
          "grupo": "Color",
          "opciones": [
            { "valor": "Negro mate", "sku": "NEL-MAS-001-NBK", "agotado": false, "imagen": "assets/img/productos/p001-negro.webp" },
            { "valor": "Blanco",     "sku": "NEL-MAS-001-WHT", "agotado": false, "imagen": null }
          ]
        },
        {
          "grupo": "Presentación",
          "opciones": [
            { "valor": "Standard (4 cabezales)", "precio_delta": 0 },
            { "valor": "Pro (6 cabezales + maletín)", "precio_delta": 25000, "sku": "NEL-MAS-001-PRO" }
          ]
        }
      ],

      // ── TEXTOS COMERCIALES (vetadas: cura / elimina / garantiza / tratamiento) ───
      "descripcion_corta": "Masajeador percusivo de 6 velocidades con 4 cabezales intercambiables.",
      "beneficios": [                           // 3–5 viñetas cortas (ficha)
        "6 niveles de intensidad ajustable",
        "Batería recargable hasta 6 horas de uso",
        "Incluye 4 cabezales para diferentes zonas",
        "Operación silenciosa (<45 dB)"
      ],
      "descripcion_larga": "<p>…pestaña Descripción…</p>",
      "caracteristicas": { "potencia": "30 W", "velocidades": "6", "bateria": "2000 mAh", "peso": "680 g", "garantia": "6 meses" },
      "modo_uso": "Aplicar sobre la zona deseada 1 a 2 minutos por sesión…",   // pestaña Modo de uso

      // ── SOCIAL ─────────────────────────────────────────────────────
      "rating": 4.6, "reseñas_count": 32,

      // ── ESTADOS DEL PRODUCTO ───────────────────────────────────────
      "stock": true,                            // ✔ false → CTA cambia a "Avísame cuando llegue" (hist. 2.5)
      "destacado": true,                        // más vendidos / inicio
      "nuevo": false,
      "fecha_ingreso": "2026-09-20",

      // ── NORMATIVO ──────────────────────────────────────────────────
      // ✔ RENOMBRADO invima → registroSanitario. null u omitido = el bloque NO aparece en la ficha.
      "registroSanitario": {
        "tipo": "Registro Sanitario",           // | "Notificación Sanitaria Obligatoria" | "Certificado de calidad"
        "numero": "SOCE000123-2026",
        "entidad": "INVIMA",                    // por si a futuro se venden en otros países/entidades
        "titular": "Distribuidora Nacional S.A.S.",
        "enlace_verificacion": "https://visiomed.invima.gov.co/…"  // opcional, link verificable = confianza
      }
    }
  ]
}
```

## 2. Tabla de cambios v1.0 → v2.0

| Campo | Estado | Regla |
|---|---|---|
| `sku` | ✅ confirmado | Obligatorio. Va literal en el mensaje WA (`Referencia:`). Si una variante define su propio `sku`, ese reemplaza al del producto en el mensaje. |
| `variantes` | 🆕 | Array de `{grupo, opciones[]}`. Opciones admiten `sku`, `precio_delta`, `imagen`, `agotado`. El precio mostrado = `precio + Σ precio_delta activos`. La selección vive solo en memoria (no hay carrito). |
| `video` | 🆕 | Opcional. Con `preload="none"` + poster → cero costo de red hasta el play (evento #12). |
| `registroSanitario` | 🔄 renombrado desde `invima` | Condicional: la plantilla lo renderiza **solo** si existe y `numero` no vacío. Nunca se inventa. |
| `stock` | ✅ | Booleano simple (dropshipping: `false` = agotado temporal, la ficha sigue pública para captar `restock_alert`). |
| `destacado` / `oferta` / `etiquetas` | ✅ | Sin cambios; alimentan inicio, `/ofertas`, búsqueda y relacionados. |

## 3. Contrato con `whatsapp.js` (campos que toca cada mensaje)

```
buildOrderMessage()   ← nombre, sku(+variante), precio(+Σ deltas), slug→productUrl con UTM
buildRestockMessage() ← nombre, sku, slug
renderBuyBox()        ← lee stock para elegir mensaje/label/color del botón
```

## 4. Validaciones nuevas para `tools/validate-data.js`

1. En cada variante: `sku` único globalmente; `precio_delta ≥ 0`; si todas las opciones de un grupo están `agotado:true` → warn (debería ser `stock:false`).
2. `video.mp4` presente ⇒ debe existir `video.poster`.
3. `registroSanitario.numero` con regex por tipo (p. ej. `SOCE\d{6}-\d{4}`) y `enlace_verificacion` https.
4. Palabras vetadas revisadas también dentro de `variantes[].valor` y `modo_uso`.
5. Advertencia dura si `oferta:true` pero `precio_anterior` es null (badge −% imposible).
