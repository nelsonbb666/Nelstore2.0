# NELSTORE — Estructura de carpetas y archivos

```
nelstore/
│
├── index.html                      # 1. INICIO
├── ofertas.html                    # 5. OFERTAS (usa la lógica de categoría con filtro oferta=true)
├── busqueda.html                   # 3. RESULTADOS DE BÚSQUEDA (misma cuadrícula)
├── sobre-nosotros.html             # 6
├── contacto.html                   # 7
├── faq.html                        # 8
├── 404.html                        # 10
│
├── categorias/                     # 2. PLANTILLA ÚNICA de categoría
│   ├── categoria.html              #    plantilla base (?cat=slug)
│   ├── masajeadores-recuperacion.html   # páginas "físicas" (SEO) → cargan la misma plantilla
│   ├── soporte-articulaciones.html
│   ├── fitness-deporte.html
│   └── cuidado-personal.html       # nuevas categorías: solo se agrega aquí + categories.json
│
├── producto/                       # 4. PLANTILLA ÚNICA de ficha
│   ├── producto.html               #    lee ?id= o slug desde products.json
│   └── (las URLs amigables se resuelven por rewrite en el servidor o
│        por JS leyendo el slug → id; NO se duplica código por producto)
│
├── components/                     # PARCIALES HTML (se inyectan 1 vez con data-include)
│   ├── topbar.html                 # C1 barra de anuncios
│   ├── header.html                 # C2+C3+C4 encabezado + menú categorías + buscador
│   ├── footer.html                 # C5 pie completo
│   └── whatsapp-float.html         # C7 botón flotante
│
├── data/                           # CATÁLOGO CENTRALIZADO (fuente única de verdad)
│   ├── products.json               # catálogo completo (esquema en docs/05)
│   ├── categories.json             # slugs, nombres, descripciones, ícono, orden del menú
│   └── site.json                   # teléfono WhatsApp, correo, horario, envío base, redes, banners
│
├── css/
│   ├── variables.css               # paleta (azul principal/blanco + acento turquesa), tipografía, espaciados, breakpoints
│   ├── base.css                    # reset, tokens, utilidades, tipografía mobile-first
│   ├── layout.css                  # grid de página, contenedores, header/footer, drawers
│   ├── components.css              # product-card, tabs, gallery, filtros, badges, carrusel, botones
│   └── pages.css                   # estilos específicos de inicio, ficha, legales, 404
│
├── js/
│   ├── app.js                      # bootstrap: inyecta componentes, init global, helpers (fetch JSON cacheado, formato COP, normalizador de texto)
│   ├── whatsapp.js                 # 🔴 S2: motor único wa.me — buildWaLink/mensajes prellenados/buy-box/sticky CTA/track() (docs/07)
│   ├── catalog.js                  # capa de datos: getProducts/getByCategory/search (sin tildes/mayúsculas)/getOffers/relacionados/bySlug
│   ├── meta.js                     # 🔴 S2: title/og:/canonical/schema.org dinámicos de la ficha
│   ├── search.js                   # C4 autocompletado del buscador
│   ├── filters.js                  # C9 filtros + orden + estado en querystring
│   ├── grid.js                     # C6/C18 render de cuadrícula + "ver más"
│   ├── variants.js                 # 🔴 S2: chips de variantes → re-render del CTA con variante en el mensaje
│   ├── pages/
│   │   ├── home.js                 # banner, secciones (destacados/más vendidos/nuevos/ofertas)
│   │   ├── category.js             # plantilla categoría (reutilizada por ofertas y búsqueda)
│   │   ├── product.js              # ficha: galería, tabs, INVIMA condicional, mensaje WA prellenado, relacionados (docs/09)
│   │   └── offers.js               # wrapper de category.js + contador regresivo opcional
│   └── components/
│       ├── product-card.js         # C6 · secundaria "Ver producto" (la compra NO arranca aquí)
│       ├── breadcrumbs.js          # C8
│       ├── gallery.js              # C11 zoom + miniaturas lazy
│       ├── video-lazy.js           # 🔴 S2: poster → play asigna src (preload none)
│       ├── carousel.js             # C12
│       ├── tabs.js                 # C10 (pestañas desktop / acordeón móvil)
│       ├── star-rating.js          # C13
│       ├── sticky-cta.js           # 🔴 S2: barra fija inferior móvil en fichas
│       ├── trust-block.js          # 🔴 S2: confianza estandarizada desde site.json (ficha + footer)
│       └── faq.js                  # C15
│
├── assets/
│   ├── img/
│   │   ├── logo.svg                # logo NELSTORE (azul + acento)
│   │   ├── icons/                  # SVG inline refs: confianza, categorías, UI
│   │   ├── banners/                # webp optimizado, altura fija (avita CLS)
│   │   └── productos/              # [sku]-1.webp ... imágenes 1:1 comprimidas (≤60 KB) + placeholder.svg
│   ├── video/                      # 🔴 S2 mp4 de "en uso" ≤15 s, <2 MB, poster webp (lazy al play)
│   └── fonts/                      # woff2 subsets (opcional; fallback sistema)
│
├── docs/                           # este entregable de estructura
│   ├── 01-mapa-sitio.md
│   ├── 02-wireframes.md
│   ├── 03-componentes.md
│   ├── 04-estructura-carpetas.md
│   ├── 05-esquema-products-json.md      # v1 (histórico)
│   ├── 06-flujo-usuario.md              # 🔴 S2: diagrama llegada → WhatsApp
│   ├── 07-whatsapp-link-function.md     # 🔴 S2: js/whatsapp.js completo
│   ├── 08-eventos-medicion.md           # 🔴 S2: eventos GA4/Pixel + UTM
│   ├── 09-plantilla-ficha-producto.md   # 🔴 S2: ficha HTML/CSS/JS
│   └── 10-products-json-v2-sprint2.md   # 🔴 S2: esquema v2.0 (variantes, video, registroSanitario)
│
├── .htaccess                       # (si se hospeda en Apache) rewrites /producto/slug, cache, gzip
├── sitemap.xml / robots.txt        # SEO
└── README.md                       # cómo agregar un producto/categoría sin tocar diseño
```

## Decisiones clave

| Decisión | Motivo |
|---|---|
| Parciales en `components/` + `data-include` | Header/footer definidos **una sola vez**; sin build tools. Con `.htaccess` se puede pre-renderizar si se requiere SEO estricto. |
| `categories.json` separado | El menú, las páginas de categoría y los breadcrumbs se generan de él → **categorías nuevas sin rediseño**. |
| `site.json` para datos de negocio | Número de WhatsApp, tarifas de envío y horarios cambian seguido: un solo archivo, cero riesgo de inconsistencia. |
| Búsqueda y ofertas = variantes de la plantilla de categoría | Una sola cuadrícula/filtro implementado (`category.js`), menos código y UX idéntica. |
| Imágenes `.webp` con fallback `.jpg`, `loading="lazy"` | Carga rápida en móviles de gama media (mercado objetivo CO). |
| Estado de filtros en querystring | Resultados compartibles por WhatsApp y mejor indexación. |


---

## Actualización Sprint 2 — módulos nuevos y por qué

| Módulo | Regla del sprint que cubre |
|---|---|
| `js/whatsapp.js` | Fuente única de enlaces wa.me, mensaje prellenado con sku+precio+enlace+variante, estados disponible/agotado, medición (docs/07) |
| `js/variants.js` + `components/sticky-cta.js` | Variante reflejada en el mensaje + CTA siempre visible en celular |
| `components/video-lazy.js` | Video en uso con carga diferida real (`preload="none"`) |
| `components/trust-block.js` | Bloque de confianza idéntico en fichas/footer desde `site.json` (historia 2.6) |
| `js/meta.js` | SEO dinámico por ficha (title/og/canonical) para pauta en redes |
| `assets/video/` | Archivos ligeros de video por producto |

**No cambió nada de la estructura de Etapa 1**: no hay carrito, checkout ni cuentas; la plantilla de ficha y la de categoría siguen siendo únicas.
