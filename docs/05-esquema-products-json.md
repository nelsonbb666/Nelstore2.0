# NELSTORE — Esquema de `data/products.json` (y archivos de datos complementarios)

## 1. `data/products.json` — catálogo centralizado

```jsonc
{
  "meta": {
    "version": "1.0",
    "moneda": "COP",
    "actualizado": "2026-10-09"
  },
  "productos": [
    {
      // ── IDENTIFICACIÓN ─────────────────────────────────────────────
      "id": "p001",                          // único, estable (clave para la plantilla ?id=)
      "sku": "NEL-MAS-001",                  // referencia visible; va en el mensaje de WhatsApp
      "slug": "pistola-de-masaje-pro-x",     // URL amigable /producto/{slug}
      "nombre": "Pistola de Masaje Pro X 6 Velocidades",   // máx. ~70 caracteres (2 líneas en card)

      // ── CLASIFICACIÓN ──────────────────────────────────────────────
      "categoria": "masajeadores",           // slug → debe existir en categories.json
      "etiquetas": ["masajeador", "pistola", "fascitis", "recuperación", "muscular"],
                                             // buscables; incluir variantes sin tilde p. ej. "recuperacion"

      // ── PRECIO ─────────────────────────────────────────────────────
      "precio": 129900,                      // entero en COP, sin puntos ni símbolo
      "precio_anterior": 179900,             // null si no hay descuento; % se calcula al vuelo
      "oferta": true,                        // aparece en /ofertas y sección "Ofertas del día"
      "oferta_hasta": "2026-10-15",          // null si es permanente; alimenta el contador

      // ── IMÁGENES (cuadradas 1:1, webp optimizado ≤60 KB) ───────────
      "imagenes": [
        "assets/img/productos/p001-1.webp",  // [0] = principal (card + meta og:image)
        "assets/img/productos/p001-2.webp",
        "assets/img/productos/p001-3.webp"
      ],
      "imagen_alt": "Pistola de masaje negra con 4 cabezales",  // accesibilidad + SEO

      // ── TEXTOS COMERCIALES (sin afirmaciones médicas) ───────────────
      "descripcion_corta": "Masajeador percusivo de 6 velocidades con 4 cabezales intercambiables.",
      "beneficios": [                        // bullets de la ficha (máx. 5)
        "6 niveles de intensidad ajustable",
        "Batería recargable hasta 6 horas de uso",
        "Incluye 4 cabezales para diferentes zonas",
        "Operación silenciosa (<45 dB)"
      ],
      "descripcion_larga": "<p>HTML o texto plano para la pestaña Descripción…</p>",
      "caracteristicas": {                   // tabla de especificaciones (pestaña)
        "potencia": "30 W",
        "velocidades": "6",
        "bateria": "2000 mAh",
        "peso": "680 g",
        "garantia": "6 meses"
      },
      "modo_uso": "Aplicar sobre la zona deseada durante 1 a 2 minutos por sesión…",

      // ── CALIFICACIÓN SOCIAL ──────────────────────────────────────────
      "rating": 4.6,                         // 0–5, una cifra decimal
      "reseñas_count": 32,

      // ── STOCK Y VISIBILIDAD ──────────────────────────────────────────
      "stock": true,                         // false → filtro "disponibilidad" la oculta/marca
      "destacado": true,                     // secciones "Más vendidos"/"Recién llegados" del inicio
      "nuevo": false,                        // badge "Nuevo"
      "fecha_ingreso": "2026-09-20",         // orden de "Recién llegados"

      // ── NORMATIVO (CONDICIONAL) ──────────────────────────────────────
      "invima": {                            // null u omitido si el producto NO lo requiere
        "tipo": "Registro Sanitario",        // | "Notificación Sanitaria Obligatoria"
        "numero": "SOCE000123-2026",
        "titular": "Distribuidora Nacional S.A.S."
      }
    }
    // … resto de productos
  ]
}
```

### Reglas del esquema

| Campo | Tipo | Obligatorio | Notas |
|---|---|---|---|
| `id`, `sku`, `slug`, `nombre`, `categoria`, `precio`, `imagenes` | — | ✅ | `slug` único; `categoria` valida contra `categories.json` |
| `precio_anterior` | number\|null | ➖ | Si es ≠ null, la card muestra badge `−X%` calculado |
| `etiquetas` | string[] | ✅ | Base de la búsqueda junto con nombre/descripción/categoría; guardar versiones **sin tilde** para matching robusto |
| `invima` | object\|null | ➖ | La ficha solo renderiza el bloque si existe y `numero` no está vacío |
| `stock` | boolean | ✅ | Dropshipping: `false` = "bajo pedido (5–10 días)", no desaparece la ficha |
| Búsqueda normalizada | — | — | El JS hace `toLowerCase()` + `.normalize('NFD').replace(/\p{Diacritic}/gu,'')` sobre nombre+descripción+categoría+etiquetas |

## 2. `data/categories.json` — hace extensible la navegación

```jsonc
[
  {
    "slug": "masajeadores",
    "nombre": "Masajeadores y recuperación",
    "descripcion": "Equipos para el cuidado muscular y la recuperación post-entrenamiento.",
    "icono": "assets/img/icons/cat-masajeadores.svg",
    "imagen": "assets/img/productos/cat-masajeadores.webp",
    "orden": 1
  },
  { "slug": "soporte", "nombre": "Soporte y articulaciones", "...": "…" },
  { "slug": "fitness", "nombre": "Fitness y deporte", "...": "…" },
  { "slug": "cuidado", "nombre": "Cuidado personal", "...": "…" }

  // Para agregar "Suplementos" o "Cuidado de piel":
  // 1) nuevo objeto aquí  2) página física en /categorias copiando la plantilla. Nada más.
]
```

## 3. `data/site.json` — datos de negocio editables sin tocar código

```jsonc
{
  "marca": "NELSTORE",
  "whatsapp": {
    "numero": "573001234567",                 // formato wa.me (código país + número)
    "horario": "Lunes a sábado, 8:00 a.m. – 8:00 p.m.",
    "mensaje_base": "Hola NELSTORE 👋, quiero información sobre:"
    // la ficha arma: mensaje_base + " {nombre} (Ref: {sku}) — ${precio formateado}"
  },
  "correo": "hola@nelstore.co",
  "envio": {
    "base_cop": 9900,
    "gratis_desde_cop": 150000,
    "cobertura": "Todo Colombia",
    "entrega_dias": "2 a 5 días hábiles"
  },
  "pago_contra_entrega": true,
  "redes": { "instagram": "https://instagram.com/nelstore.co", "tiktok": "…" },
  "banners": [
    { "imagen": "assets/img/banners/slide1.webp", "titulo": "Recupera tu cuerpo", "enlace": "categorias/masajeadores-recuperacion.html" }
    // 2–3 slides configurables
  ]
}
```

## 4. Validación mínima propuesta (script `tools/validate-data.js`, etapa siguiente)

- IDs, SKUs y slugs únicos · `categoria` existente en categories.json · `precio > 0` ·
- `precio_anterior > precio` cuando exista · imágenes con ruta existente ·
- vetting de textos: lista de palabras prohibidas (`cura`, `elimina`, `garantizado`, `tratamiento de`, …) que lance warning al construir.
