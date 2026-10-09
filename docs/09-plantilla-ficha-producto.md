# NELSTORE — Sprint 2 · Plantilla de ficha de producto (`producto/producto.html` + `css` + `js/pages/product.js`)

Plantilla **única** para todas las fichas: se alimenta de `data/products.json` según el `slug`
(resuelto por rewrite en `.htaccess` o por `?id=` como fallback). No hay HTML duplicado por producto.

Estructura mobile-first. En celular: galería → precio/compra → confianza → pestañas → relacionados,
con **barra fija inferior** con el CTA. En desktop: dos columnas (galería izquierda, compra derecha).

---

## 1. HTML — `producto/producto.html`

```html
<!DOCTYPE html>
<html lang="es-CO">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1">
  <!-- title / og: / canonical los completa product.js desde products.json (SEO dinámico) -->
  <title>NELSTORE — Ficha de producto</title>
  <link rel="preload" as="image" href="assets/img/productos/placeholder.webp">
  <link rel="stylesheet" href="css/variables.css">
  <link rel="stylesheet" href="css/base.css">
  <link rel="stylesheet" href="css/components.css">
  <link rel="stylesheet" href="css/pages.css">
</head>
<body class="page-product">

  <!-- Componentes compartidos (se inyectan 1 sola vez, ver components/) -->
  <div data-include="../components/topbar.html"></div>
  <div data-include="../components/header.html"></div>

  <main id="contenido">
    <nav aria-label="Ruta de navegación" class="breadcrumbs" id="breadcrumbs"></nav>

    <article class="ficha" itemscope itemtype="https://schema.org/Product">

      <!-- ══ COLUMNA 1 · MEDIO ══════════════════════════════════════ -->
      <section class="ficha-medio" aria-label="Galería del producto">
        <div class="gallery" id="gallery">
          <div class="gallery-main" id="gallery-main" tabindex="0"
               aria-label="Imagen principal, pasa el cursor o pellizca para ampliar">
            <img id="gallery-img" src="" alt="" loading="lazy" decoding="async">
            <span class="zoom-hint" aria-hidden="true">🔍</span>
          </div>
          <div class="gallery-thumbs" role="tablist" aria-label="Miniaturas" id="gallery-thumbs">
            <!-- botones <img> generados por gallery.js desde p.imagenes -->
          </div>
        </div>

        <!-- Video en uso · lazy: solo se descarga al pulsar (historia 2.1) -->
        <div class="video-block" id="video-block" hidden>
          <button class="video-poster" id="video-play" type="button"
                  aria-label="Reproducir video del producto en uso">
            <img id="video-poster-img" src="" alt="" loading="lazy">
            <span class="play-badge" aria-hidden="true">▶ Ver en uso</span>
          </button>
          <video id="product-video" controls preload="none" playsinline></video>
        </div>
      </section>

      <!-- ══ COLUMNA 2 · COMPRA ═════════════════════════════════════ -->
      <section class="ficha-compra">
        <h1 itemprop="name" id="p-nombre">—</h1>

        <div class="rating-row">
          <span class="stars" id="p-stars" aria-hidden="true"></span>
          <span class="rating-txt"><b id="p-rating">0</b> · <a href="#pestana-resenas">
            <span id="p-reseñas">0</span> reseñas</a></span>
        </div>

        <div class="price-block">
          <p class="price-now"  id="p-precio">—</p>
          <p class="price-was"  id="p-anterior" hidden></p>
          <span class="badge-off" id="p-descuento" hidden></span>
        </div>

        <p class="desc-short" id="p-desc-corta" itemprop="description"></p>

        <!-- Beneficios: 3–5 viñetas cortas, sin afirmaciones médicas -->
        <ul class="beneficios" id="p-beneficios"></ul>

        <!-- Variantes (color / talla / presentación) → entran al mensaje de WhatsApp -->
        <div class="variantes" id="p-variantes" role="group" aria-label="Elige una opción">
          <!-- grupos generados por product.js: <fieldset><legend>Color</legend>
               <button class="variant-chip" data-group="Color" data-value="Azul">… -->
        </div>

        <!-- Stock visible ANTES de escribir (historia 2.5) -->
        <p class="stock-note ok"   id="p-stock-ok"   hidden>✔ Disponible — listo para despacho</p>
        <p class="stock-note alert" id="p-stock-out"  hidden>Agotado temporalmente · te avisamos cuando llegue</p>

        <!-- INVIMA condicional: product.js solo lo muestra si p.invima existe -->
        <div class="invima" id="p-invima" hidden>
          <strong id="invima-tipo"></strong> <span id="invima-numero"></span>
          <p class="invima-titular" id="invima-titular"></p>
        </div>

        <!-- CAJA DE COMPRA: #buy-box lo rellena NELWA.renderBuyBox().
             El <noscript> garantiza que SIN JS también haya enlace a WhatsApp. -->
        <div id="buy-box" class="buy-box"></div>
        <noscript>
          <a class="btn-wa-main" target="_blank" rel="noopener"
             href="https://wa.me/57NUMERO?text=Hola%20Nelstore%2C%20quiero%20pedir%20el%20producto%20de%20esta%20p%C3%A1gina.%20Referencia%20y%20precio%20seg%C3%BAn%20la%20ficha.">
            Pedir por WhatsApp</a>
        </noscript>

        <!-- Bloque de confianza (historia 2.6) -->
        <ul class="confianza-list">
          <li>🚚 Envío a toda Colombia — <span data-site="envio.entrega_dias"></span></li>
          <li>💵 Pago contra entrega disponible cuando aplique</li>
          <li>↩️ Cambios y devoluciones — <a href="../legales/cambios-devoluciones.html">ver condiciones</a></li>
          <li>💬 Atención humana por WhatsApp: <span data-site="whatsapp.horario"></span></li>
        </ul>
      </section>
    </article>

    <!-- ══ PESTAÑAS (acordeón en celular) ═══════════════════════════ -->
    <section class="tabs-zone" aria-label="Información detallada">
      <div class="tabs" id="p-tabs">
        <button role="tab" aria-selected="true"  data-tab="desc">Descripción</button>
        <button role="tab" aria-selected="false" data-tab="specs">Características</button>
        <button role="tab" aria-selected="false" data-tab="uso">Modo de uso</button>
        <button role="tab" aria-selected="false" data-tab="env">Envíos y devoluciones</button>
      </div>
      <div role="tabpanel" data-panel="desc" id="p-tab-desc"></div>
      <div role="tabpanel" data-panel="specs" hidden><table id="p-specs-table"></table></div>
      <div role="tabpanel" data-panel="uso"   hidden id="p-tab-uso"></div>
      <div role="tabpanel" data-panel="env"   hidden id="p-tab-env">
        <!-- texto estándar desde site.json: cobertura, días, costo base, gratis desde,
             contra entrega, cambios 5 días / devoluciones según legales -->
      </div>
    </section>

    <!-- ══ RELACIONADOS (misma ProductCard, misma plantilla) ════════ -->
    <section class="relacionados" aria-label="Productos relacionados">
      <h2>También puede interesarte</h2>
      <div class="grid grid-scroll" id="p-relacionados"></div>
    </section>
  </main>

  <!-- Barra fija inferior SOLO celular: siempre tiene el CTA a la vista -->
  <div class="sticky-cta" id="sticky-cta" aria-label="Comprar por WhatsApp"></div>

  <div data-include="../components/whatsapp-float.html"></div>
  <div data-include="../components/footer.html"></div>

  <script src="js/app.js" defer></script>       <!-- bootstrap + data-include + helpers -->
  <script src="js/catalog.js" defer></script>   <!-- capa de datos products.json -->
  <script src="js/whatsapp.js" defer></script>  <!-- motor wa.me (docs/07) -->
  <script src="js/pages/product.js" defer></script>
</body>
</html>
```

## 2. CSS — extractos clave (`css/pages.css`, tokens en `variables.css`)

```css
/* Paleta: --azul:#1456d7 · --blanco:#fff · --tinta:#16233b · --turquesa:#12b5a5 */

.ficha { display: grid; gap: 20px; padding: 12px; }                 /* MOBILE FIRST */
@media (min-width: 900px) {                                          /* desktop 2 col */
  .ficha { grid-template-columns: minmax(0,520px) 1fr; gap: 40px; }
}

.gallery-main img { aspect-ratio: 1/1; width: 100%; object-fit: cover; border-radius: 12px; }
.gallery-main.zooming img { transform: scale(2); transform-origin: var(--zx) var(--zy); }
.gallery-thumbs button.is-active { outline: 2px solid var(--azul); }

.video-poster { position: relative; display: block; width: 100%; }   /* <video preload="none"
   no descarga nada hasta pulsar → play() setea src = dataset.src (lazy real) */

.price-now { font-size: 1.75rem; font-weight: 800; color: var(--tinta); }
.price-was { text-decoration: line-through; color: #7a8699; }
.badge-off { background: #e8f7f5; color: var(--turquesa); font-weight: 700; }

.btn-wa-main {                       /* único botón de acción de compra */
  display: flex; align-items: center; justify-content: center; gap: 10px;
  min-height: 56px; width: 100%; border-radius: 14px;            /* táctil ≥48px */
  background: #25D366; color: #fff; font-weight: 700; font-size: 1.05rem;
  text-decoration: none; box-shadow: 0 6px 16px rgba(37,211,102,.35);
}
.btn-wa-alert { background: var(--azul); box-shadow: none; }      /* agotado */

.sticky-cta {                        /* fijo inferior, solo <900px */
  position: fixed; inset-inline: 0; bottom: 0; z-index: 60;
  display: flex; gap: 12px; align-items: center; padding: 10px 12px;
  background: #fff; border-top: 1px solid #e6eaf2;
  box-shadow: 0 -4px 14px rgba(22,35,59,.08);
}
.sticky-cta .price-now { font-size: 1.1rem; flex: 0 0 auto; }
.sticky-cta .btn-wa-main { flex: 1; }
body.page-product { padding-bottom: 84px; }                        /* hueco del CTA fijo */
@media (min-width: 900px) { .sticky-cta { display: none; } body.page-product { padding-bottom: 0; } }

.tabs [role="tab"][aria-selected="true"] { color: var(--azul); border-bottom: 3px solid var(--azul); }
@media (max-width: 600px) { .tabs { display: none; }              /* en celular = acordeón
   (tabs.js renderiza ambos patrones según breakpoint, mismo contenido) */ }
```

## 3. JS — `js/pages/product.js` (lógica, comentada)

```js
/* Resuelve el producto (slug de la URL o ?id=), pinta la ficha completa desde
   products.json y conecta variantes/galería/video/CTA. Sin frameworks. */
(function () {
  'use strict';

  function getP() {                                   // slug → /producto/{slug} o ?id=
    var m = location.pathname.match(/\/producto\/([^/?]+)/);
    return m ? NELCATALOG.getBySlug(m[1]) : NELCATALOG.getById(new URLSearchParams(location.search).get('id'));
  }

  document.addEventListener('nel:ready', function () {         // app.js terminó data-include
    NELWA.init(function () {
      var p = getP();
      if (!p) { location.replace('/404.html'); return; }       // id inválido → 404 amigable

      /* SEO + schema.org dinámicos */
      document.title = p.nombre + ' | NELSTORE';
      NELMETA.set({ ogImage: p.imagenes[0], desc: p.descripcion_corta });

      /* Galería (lazy + zoom) */
      NELGALLERY.mount('#gallery', p.imagenes, p.imagen_alt);

      /* Video opcional: el <source> NO se asigna hasta pulsar play (historia 2.1) */
      if (p.video) {
        document.getElementById('video-block').hidden = false;
        document.getElementById('video-play').addEventListener('click', function () {
          var v = document.getElementById('product-video');
          v.src = p.video.mp4; v.poster = p.video.poster; v.hidden = false;
          this.hidden = true; v.play();
        });
      }

      /* Precio, descuento calculado, rating, beneficios */
      NELPAGE.fillPrice(p); NELPAGE.fillRating(p); NELPAGE.fillBenefits(p.beneficios);

      /* Variantes → state.variante → re-render del CTA (historia 2.4) */
      var state = { variante: null };
      NELVARIANTS.mount('#p-variantes', p.variantes, function (v) {
        state.variante = v; NELWA.renderBuyBox(p, state);
      });

      /* Estado del producto (2.5): disponible vs agotado — el CTA lo decide NELWA */
      document.getElementById(p.stock ? 'p-stock-ok' : 'p-stock-out').hidden = false;

      /* INVIMA condicional: se oculta si no existe o el número viene vacío */
      if (p.invima && p.invima.numero) NELPAGE.showInvima(p.invima);

      /* CTA principal + barra fija móvil + medición ViewContent (docs/07, docs/08) */
      NELWA.renderBuyBox(p, state);
      NELWA.track('view_content', { product_id: p.id, category: p.categoria, price_cop: p.precio });

      /* Pestañas + relacionados (misma ProductCard del catálogo) */
      NELTABS.mount('#p-tabs', p);
      NELGRID.render('#p-relacionados', NELCATOG.relatedTo(p, 6));
    });
  });
})();
```

## 4. Checklist de criterios de aceptación cubiertos

| Historia | Cómo se cumple en la plantilla |
|---|---|
| 2.1 | Galería + video lazy (`preload="none"`) + precio + beneficios + pestañas |
| 2.2 | Un solo `<a>` directo a `wa.me`, sin formularios ni pasos intermedios |
| 2.3 | `buildOrderMessage()` incluye nombre, sku, precio y enlace |
| 2.4 | Chips de variante re-pintan el `href` con la variante en el mensaje |
| 2.5 | Nota de stock visible + botón alternativo “Avísame cuando llegue” |
| 2.6 | `confianza-list` + pestaña “Envíos y devoluciones” desde `site.json` |
| 2.7 | `bindClickTracking()` → evento con producto, categoría, origen y UTM |
| 2.8 | Todo se lee de `products.json`; editar el JSON cambia ficha, card y relacionados |
| Accesibilidad | Botones ≥56 px, `aria-label`, roles tab/tabpanel, foco visible, alt desde JSON |
| Sin JS | `<noscript>` con anchor wa.me real; imágenes con `loading="lazy"` WebP |
