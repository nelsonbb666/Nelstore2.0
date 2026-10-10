# NELSTORE — Sprint 2 · Eventos de medición (GA4 + Meta Pixel, sin librerías pesadas)

## 1. Arquitectura de medición

```
Clic / vista en la web
      │
      ▼
NELWA.track(evento, payload)          ← único punto de emisión (js/whatsapp.js)
      │
      ├──► window.dataLayer.push()    → GTM → GA4 (eventos personalizados)
      └──► window.fbq('track', …)     → Meta Pixel (estándar: ViewContent, Contact)
```

- **No se carga ningún SDK propio**: `dataLayer` y `fbq` pueden no existir; `track()` los envuelve
  en `try/catch` y nunca rompe el flujo de compra.
- El clic **no hace `preventDefault`**: primero se empuja el evento a `dataLayer` (síncrono) y el
  navegador abre `wa.me`. Para asegurar la llegada a GA4 se recomienda además
  `navigator.sendBeacon` en una fase 2 si se detecta pérdida de eventos.
- Los payloads incluyen siempre: `product_id`, `sku`, `category`, `price_cop`, `source_page`,
  `utm` (los 5 parámetros heredados de la URL de entrada).

## 2. Catálogo de eventos

### A. Prioridad máxima (la métrica del negocio)

| # | Evento interno | GA4 (vía GTM) | Meta Pixel | Se dispara cuando… | Payload |
|---|---|---|---|---|---|
| 1 | `whatsapp_click` (intent:`order`) | custom `whatsapp_click` | **`Contact`** | Clic en “Pedir por WhatsApp” (ficha o barra fija) | product_id, sku, category, price_cop, variant, source_page, utm |
| 2 | `whatsapp_click` (intent:`restock_alert`) | custom `whatsapp_restock_click` | `Contact` | Clic en “Avísame cuando llegue” (agotado) | product_id, sku, category, source_page, utm |
| 3 | `whatsapp_click` (intent:`general`) | custom `whatsapp_general_click` | `Contact` | Botón flotante, header, contacto, footer, 404 | context(page), source_page, utm |
| 4 | `view_content` | `view_item` | **`ViewContent`** (value+currency COP) | Ficha renderizada con producto válido | product_id, category, price_cop |

### B. Descubrimiento e intención

| # | Evento interno | Equivalente GA4 | Se dispara cuando… | Payload |
|---|---|---|---|---|
| 5 | `search` | `search` | Submit del buscador | query_normalizada, results_count |
| 6 | `search_select` | custom | Usuario elige un ítem del autocompletado | query, product_id/slug destino |
| 7 | `view_category` | custom `view_category` | Plantilla de categoría cargada | category_slug, products_count |
| 8 | `view_offers` | custom | Página `/ofertas` cargada | products_count |
| 9 | `filter_apply` | custom | Cambio de precio/rating/disponibilidad/orden | category, filtros_aplicados, results_count |
| 10 | `load_more` | custom | Botón “Ver más” del grid | category, page |
| 11 | `carousel_slide` *(opcional, throttle)* | custom | Slide visible >2 s en banner | slide_index, enlace |

### C. Confianza y contenido

| # | Evento interno | Equivalente GA4 | Se dispara cuando… |
|---|---|---|---|
| 12 | `video_play` | `video_start` | Pulsar “Ver en uso” (lazy cumplido = señal de interés real) |
| 13 | `tab_open` | custom | Abrir características / modo de uso / envíos |
| 14 | `gallery_zoom` | custom | Activar zoom en imagen principal |
| 15 | `variant_select` | custom | Elegir color/talla/presentación (predice calidad del clic a WA) |
| 16 | `scroll_depth_50/75/95` | informe de desplazamiento nativo GA4 | Umbrales de scroll |
| 17 | `page_view` | nativo GA4 | Todas (automático con gtag/GTM) |

### D. Post-sitio (fuera del alcance técnico, pero se registran para el dashboard)

| Métrica de negocio | Cómo se obtiene |
|---|---|
| Pedidos confirmados / rechazados (contra entrega) | WhatsApp Business + registro manual del asesor, referenciando el `sku` que llega en el mensaje |
| Tasa de cierre | pedidos confirmados ÷ evento #1 |
| Costo por pedido | inversión en pauta ÷ pedidos por `utm_source` |

## 3. Parámetros UTM (obligatorios en toda pauta)

```
https://nelstore.co/producto/{slug}
  ?utm_source=tiktok|instagram|google|facebook
  &utm_medium=cpc|organic|story|reel
  &utm_campaign={mes-campana}
  &utm_content={creativo}
```

- La web **no borra ni reescribe** los UTM; `whatsapp.js` los vuelve a adjuntar al `Enlace:` del
  mensaje prellenado → cada chat recibido trae la atribución de canal dentro del propio mensaje.
- Recomendado: campaña GA4 “canal = referrer + utm_source” para cruzar contra el evento `Contact`.

## 4. Errores comunes que este esquema evita

1. Medir con listeners que hacen `preventDefault` → se pierden clics reales. Aquí el anchor funciona solo.
2. Contar “clics al botón” sin distinguir orden vs aviso vs consulta general → los 3 llevan `intent`.
3. Disparar `ViewContent` antes de validar el producto → se dispara solo con ficha válida (#4).
4. Varios números de WhatsApp → **un solo número** en `site.json`; todos los enlaces salen de `buildWaLink()`.

## 5. Dashboard mínimo sugerido (fase siguiente)

- Panel 1: `whatsapp_click[order]` por producto y por `utm_source` (tabla top 10).
- Panel 2: Embudo `view_content → whatsapp_click[order]` (tasa de clic = KPI primario).
- Panel 3: Productos con muchas vistas y pocos clics → candidatos a bajar precio o cambiar foto.
- Panel 4: `restock_alert` → demanda retenida para reimportar.
