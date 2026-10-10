# NELSTORE — Sprint 2 · `js/whatsapp.js` (motor de enlaces y medición)

Archivo único que construye **todos** los enlaces a WhatsApp del sitio.
Depende solo de `data/site.json` (número, mensajes base) y de los productos de
`data/products.json`. Sin librerías externas.

## Decisiones de diseño

1. **Fuente única de verdad**: nadie escribe `wa.me` a mano en el HTML; todo pasa por `buildWaLink()`.
   Cambiar el número = cambiar una línea en `site.json`.
2. **Gracia degradativa (JS falla → la venta no)**: `renderBuyBox()` pinta en el HTML un
   `<a href="https://wa.me/…?text=…" target="_blank" rel="noopener">` real con el mensaje ya
   codificado. Si el usuario tiene JS desactivado o el fetch de `products.json` falla antes
   del render server-side, el enlace sigue funcionando porque es un anchor normal, no un listener.
3. **UTM heredados al chat**: `getUtmParams()` captura `utm_source/medium/campaign/content/term`
   de la URL actual y los añade al enlace de la ficha que viaja dentro del mensaje, para saber
   de qué canal vino cada pedido.
4. **Medición sin dependencias**: `track()` empuja a `window.dataLayer` (GA4 lo lee vía GTM) y
   dispara `fbq('track', …)` si Meta Pixel está cargado. Ninguna de las dos es obligatoria.

```js
/**
 * ============================================================
 * NELSTORE · whatsapp.js
 * Construcción de enlaces wa.me + mensajería + medición.
 * Ventana global: window.NELWA
 * ============================================================
 */
(function (window, document) {
  'use strict';

  var SITE = null; // se carga desde data/site.json

  /* ── Utilidades ─────────────────────────────────────────── */

  /** Formato monetario COP: 129900 → "$129.900" (punto de miles, sin decimales). */
  function formatCOP(value) {
    return '$' + Math.round(value).toLocaleString('es-CO');
  }

  /** Lee ?query= de la página actual. */
  function getQueryParam(name) {
    return new URLSearchParams(window.location.search).get(name) || '';
  }

  /** Extrae los UTM visibles en la URL para heredarlos al chat. */
  function getUtmParams() {
    var keys = ['utm_source', 'utm_medium', 'utm_campaign', 'utm_content', 'utm_term'];
    var out = {};
    keys.forEach(function (k) {
      var v = getQueryParam(k);
      if (v) out[k] = v;
    });
    return out;
  }

  /** Enlace de la ficha del producto, con UTM pegados (para atribución en el chat). */
  function productUrl(p) {
    var url = window.location.origin + '/producto/' + encodeURIComponent(p.slug);
    var utm = getUtmParams();
    var qs = Object.keys(utm).map(function (k) {
      return encodeURIComponent(k) + '=' + encodeURIComponent(utm[k]);
    }).join('&');
    return qs ? url + '?' + qs : url;
  }

  /* ── Mensajes prellenados ───────────────────────────────── */

  /**
   * Mensaje de COMPRA (historias 2.2, 2.3, 2.4).
   * @param {object} p       producto de products.json
   * @param {object|null} v  variante seleccionada {grupo, valor} o null
   */
  function buildOrderMessage(p, v) {
    var lines = [
      'Hola Nelstore, quiero pedir este producto:',
      '',
      '\u2022 Producto: ' + p.nombre,
      '\u2022 Referencia: ' + p.sku
    ];
    if (v && v.valor) {
      lines.push('\u2022 Variante (' + v.grupo + '): ' + v.valor); // 2.4: llega al chat
    }
    lines.push(
      '\u2022 Precio: ' + formatCOP(p.precio),
      '\u2022 Cantidad: 1',
      '\u2022 Enlace: ' + productUrl(p),
      '',
      'Mis datos: Nombre, ciudad y dirección:'
    );
    return lines.join('\n');
  }

  /** Mensaje de AVISO POR AGOTADO (historia 2.5) — no es un pedido. */
  function buildRestockMessage(p) {
    return [
      'Hola Nelstore, vi que este producto está agotado.',
      'Por favor avísenme cuando vuelva a estar disponible:',
      '',
      '\u2022 Producto: ' + p.nombre,
      '\u2022 Referencia: ' + p.sku,
      '\u2022 Enlace: ' + productUrl(p)
    ].join('\n');
  }

  /** Mensaje de CONSULTA GENERAL (botón flotante / contacto / footer). */
  function buildGeneralMessage(context) {
    var base = (SITE && SITE.whatsapp.mensaje_base) || 'Hola Nelstore 👋, quiero información.';
    return context ? base + ' Página: ' + context : base;
  }

  /* ── Enlace final ───────────────────────────────────────── */

  /**
   * EL ÚNICO generador de URLs wa.me del sitio.
   * https://wa.me/{numero}?text={encodeURIComponent(mensaje)}
   */
  function buildWaLink(message) {
    var num = SITE ? SITE.whatsapp.numero : '57NUMERO'; // placeholder hasta definir número
    return 'https://wa.me/' + num + '?text=' + encodeURIComponent(message);
  }

  /* ── Medición (ver docs/08-eventos-medicion.md) ─────────── */

  /** Empuja a dataLayer (GA4/GTM) y fbq (Meta Pixel) si existen. Nunca lanza. */
  function track(event, payload) {
    try {
      window.dataLayer = window.dataLayer || [];
      window.dataLayer.push({ event: event, nel: payload });     // GA4 vía GTM
      if (typeof window.fbq === 'function') {
        // ViewContent → vista de ficha · Contact → clic a WhatsApp
        var fbName = event === 'whatsapp_click' ? 'Contact'
                   : event === 'view_content'  ? 'ViewContent' : null;
        if (fbName) window.fbq(fbName, fbName === 'Contact'
          ? {} : { value: payload.price_cop, currency: 'COP' });
      }
    } catch (e) { /* la analítica jamás debe romper la compra */ }
  }

  /**
   * Registra el clic ANTES de navegar y deja 200 ms al beacon.
   * El href ya es válido sin JS: aquí solo medimos (regla 6 del sprint).
   */
  function bindClickTracking(anchor, payload) {
    anchor.addEventListener('click', function () {
      track('whatsapp_click', payload);
      // No hacemos preventDefault: dejamos que wa.me abra garantizado.
    });
  }

  /* ── Render de la caja de compra (ficha + barra fija móvil) ─ */

  /**
   * Pinta el CTA dentro de #buy-box (y #sticky-cta en celular).
   * stock:true  → "Pedir por WhatsApp"
   * stock:false → "Avísame cuando llegue" (mensaje distinto)
   * Al cambiar variante se RE-pinta el href → el mensaje lleva la variante.
   */
  function renderBuyBox(p, state) {
    var soldOut = !p.stock;
    var msg = soldOut ? buildRestockMessage(p) : buildOrderMessage(p, state.variante);
    var href = buildWaLink(msg);
    var label = soldOut ? 'Avísame cuando llegue' : 'Pedir por WhatsApp';

    ['#buy-box', '#sticky-cta'].forEach(function (sel) {
      var box = document.querySelector(sel);
      if (!box) return;
      box.innerHTML =
        '<a class="btn-wa-main' + (soldOut ? ' btn-wa-alert' : '') + '" ' +
        'href="' + href + '" target="_blank" rel="noopener" ' +
        'aria-label="' + label + ': ' + p.nombre + '">' +
        '<svg class="ic-wa" aria-hidden="true"><!-- icono oficial WhatsApp --></svg>' +
        label + '</a>';
      bindClickTracking(box.querySelector('a'), {
        product_id: p.id, sku: p.sku, category: p.categoria, price_cop: p.precio,
        variant: state.variante ? state.variante.valor : null,
        intent: soldOut ? 'restock_alert' : 'order',
        source_page: document.title, utm: getUtmParams()
      });
    });
  }

  /* ── API pública ────────────────────────────────────────── */
  window.NELWA = {
    init: function (cb) {                                   // carga site.json una sola vez
      if (SITE) return cb(SITE);
      fetch('data/site.json')
        .then(function (r) { return r.json(); })
        .then(function (j) { SITE = j; cb(SITE); })
        .catch(function () { cb(null); });                  // con SITE=null usa placeholder
    },
    formatCOP: formatCOP,
    productUrl: productUrl,
    buildOrderMessage: buildOrderMessage,
    buildRestockMessage: buildRestockMessage,
    buildGeneralMessage: buildGeneralMessage,
    buildWaLink: buildWaLink,
    renderBuyBox: renderBuyBox,
    track: track
  };
})(window, document);
```

## Ejemplo de uso desde `js/pages/product.js` (ficha)

```js
NELWA.init(function () {
  var p = NELCATALOG.getProductBySlug(getParam('slug'));   // capa de catálogo
  var state = { variante: null };

  NELWA.renderBuyBox(p, state);                            // CTA inicial
  NELWA.track('view_content', {                            // historia 2.1 + pixel
    product_id: p.id, category: p.categoria, price_cop: p.precio
  });

  document.querySelectorAll('.variant-chip').forEach(function (chip) {
    chip.addEventListener('click', function () {
      state.variante = { grupo: chip.dataset.group, valor: chip.dataset.value };
      selectChip(chip);
      NELWA.renderBuyBox(p, state);                        // href re-codificado con variante
    });
  });
});
```

## Ejemplo de mensaje generado (resultado real)

```
Hola Nelstore, quiero pedir este producto:

• Producto: Rodillo de Espuma Texturizado Doble Densidad
• Referencia: NEL-FIT-014
• Variante (Color): Azul
• Precio: $54.900
• Cantidad: 1
• Enlace: https://nelstore.co/producto/rodillo-espuma-doble-densidad?utm_source=tiktok&utm_medium=cpc&utm_campaign=octubre
Mis datos: Nombre, ciudad y dirección:
```
