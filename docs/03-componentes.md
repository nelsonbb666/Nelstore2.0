# NELSTORE — Componentes reutilizables

Cada componente se define **una sola vez** y se inyecta en todas las páginas vía
`components.js` (fetch del partial HTML + `data-include`), evitando duplicar código.

## Índice de componentes

| # | Componente | Archivo partial | Uso en páginas | Descripción / comportamiento |
|---|---|---|---|---|
| C1 | **TopBar** (barra de anuncios) | `components/topbar.html` | Todas | 3 mensajes rotativos: envío nacional · pago contra entrega · WhatsApp. Cerrable en móvil (persiste en `localStorage`). |
| C2 | **Header** (encabezado) | `components/header.html` | Todas | Sticky. Logo, buscador grande con autocompletado, menú hamburguesa ≡ (drawer en móvil), botón WhatsApp, badge "Ofertas". En desktop muestra el menú horizontal de categorías. |
| C3 | **CategoryMenu** (menú categorías) | dentro de header | Todas | Se genera desde `categories.json` → agregar una categoría actualiza el menú automáticamente. |
| C4 | **SearchBar + Autocomplete** | `components/search.js` | Header, 404 | Debounce 250 ms, normaliza tildes/mayúsculas, sugiere nombres + categorías, navega con flechas/Enter, redirige a `busqueda.html?q=`. |
| C5 | **Footer** | `components/footer.html` | Todas | 4 bloques: marca+redes · categorías (desde JSON) · ayuda/legal · contacto+horario. Línea inferior con aviso de datos (Ley 1581). |
| C6 | **ProductCard** (tarjeta producto) | `js/components/product-card.js` (plantilla renderizada por JS) | Inicio, categoría, búsqueda, ofertas, relacionados, 404 | Imagen 1:1 lazy-load (`loading="lazy"` + placeholder), badge −X% / "Nuevo", nombre máx. 2 líneas con ellipsis, estrellas + nº reseñas, precio actual destacado, precio anterior tachado. Toda la tarjeta es enlace a la ficha. |
| C7 | **WhatsAppFloat** (botón flotante) | `components/whatsapp-float.html` | Todas | Fijo abajo-derecha (z-index alto), aria-label, abre `wa.me` con mensaje genérico; en fichas puede llevar el mensaje del producto. |
| C8 | **Breadcrumbs** (migas de pan) | `js/components/breadcrumbs.js` | Categoría, ficha, búsqueda, legales, 404 | Generado desde la ruta/JSON; incluye datos estructurados `BreadcrumbList` (SEO). |
| C9 | **FilterPanel** (filtros) | `js/components/filters.js` | Categoría, ofertas, búsqueda | Desktop: sidebar fijo. Móvil: drawer inferior con overlay. Precio (rango), calificación mín., disponibilidad, solo-ofertas, ordenar por. Estado en querystring (`?cat=&min=&max=&sort=`) → compartible/indexable. Reutiliza la misma lógica en las 3 vistas de cuadrícula. |
| C10 | **Tabs/Accordion** | `js/components/tabs.js` | Ficha de producto | Pestañas en desktop; acordeón en móvil (mismo contenido, dos presentaciones CSS/JS). |
| C11 | **Gallery** (galería con zoom) | `js/components/gallery.js` | Ficha de producto | Miniaturas + imagen principal; zoom hover (desktop) / tap-to-fullscreen (móvil); swipe entre imágenes; previsualización de la siguiente imagen cargada. |
| C12 | **Carousel** (carrusel) | `js/components/carousel.js` | Banner inicio, ofertas del día, más vendidos, relacionados | Scroll-snap nativo + flechas desktop + dots; autoplay solo en banner (pausa si el usuario interactúa o pierde foco). Sin librerías. |
| C13 | **StarRating** | helper dentro de product-card/ficha | Donde haya rating | Dibuja estrellas medias (SVG inline, sin imágenes). Recibe `rating` y `count`. |
| C14 | **TrustBadges** (íconos de confianza) | snippet reutilizable | Inicio, ficha, contacto | Grid de ícono+título+texto corto: envíos · contra entrega · WhatsApp · compra segura. SVG inline (peso cero). |
| C15 | **FaqAccordion** | `js/components/faq.js` | Inicio (resumen) y faq.html | Acordeón accesible (`<button aria-expanded>`), deep-link `#pregunta-n`. |
| C16 | **EmptyState** | snippet JS | Categoría, búsqueda, ofertas | Mensaje amable + acciones (limpiar filtros / WhatsApp). |
| C17 | **Button (CTA WA)** | clase utilitaria `.btn-wa` | Fichas, header, float | Estilo primario WhatsApp (verde oficial), estado pressed/disabled, tamaño táctil ≥48 px. |
| C18 | **LoadMore/Pagination** | dentro de filters/grid JS | Categoría, ofertas, búsqueda | Botón "Ver más" que añade lotes de 12–24 productos; actualiza contador de resultados. |

## Reglas transversales de los componentes

1. **Una sola fuente de datos:** todo lo renderizado sale de `products.json` y `categories.json`.
2. **Accesibilidad mínima:** roles ARIA en menúes/drawers/tabs, foco visible, contraste AA, targets ≥48px.
3. **Rendimiento:** CSS único global + partials ligeros; imágenes `loading="lazy"` + `decoding="async"`; JS por página con `defer`; sin jQuery ni frameworks.
4. **Idiomaticidad colombiana:** precios formateados `$ 49.900` con `Intl.NumberFormat('es-CO')`.
