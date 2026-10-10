# NELSTORE — Sprint 2 · Diagrama de flujo del usuario (vitrina → WhatsApp)

## Principio rector

> **La web no vende: prepara la venta.** Su trabajo es que el cliente vea el producto,
> entienda qué es, confíe en la tienda y llegue a WhatsApp con ganas de pedir.
> No existe carrito, checkout, pasarela de pago ni cuentas de usuario.
> El único botón de acción de compra es **“Pedir por WhatsApp”**.

---

## 1. Flujo principal (happy path)

```
                 ┌──────────────────────────────────────────────┐
                 │  LLEGADA                                      │
                 │  TikTok / Instagram / anuncio Meta / Google   │
                 │  (URL con UTM → se conservan en el enlace WA) │
                 └──────────────────────┬───────────────────────┘
                                        ▼
        ┌───────────────────────────────────────────────────┐
        │  EXPLORACIÓN                                      │
        │  Inicio · Categoría · Búsqueda · Ofertas          │
        │  ProductCard: foto 1:1, nombre ≤2 líneas,         │
        │  precio COP, badge −%, "Ver producto"             │
        └──────────────────────┬────────────────────────────┘
                               ▼
        ┌───────────────────────────────────────────────────┐
        │  FICHA DE PRODUCTO  (decidir sin preguntar)       │
        │  ① Galería con zoom + miniaturas (WebP, lazy)     │
        │  ② Video en uso (lazy: carga solo al pulsar)      │
        │  ③ Precio / precio anterior / % descuento         │
        │  ④ Beneficios (3–5 viñetas cortas)                │
        │  ⑤ Selector de variante (color/talla/present.)    │
        │  ⑥ Rating + nº de reseñas                         │
        │  ⑦ Pestañas: descripción · características ·      │
        │     modo de uso · envíos y devoluciones           │
        │  ⑧ Bloque de confianza + INVIMA (si aplica)       │
        │  ⑨ Productos relacionados                         │
        └──────────────────────┬────────────────────────────┘
                               ▼
        ┌───────────────────────────────────────────────────┐
        │  CONFIANZA (antes de dar datos a un chat)         │
        │  Pago contra entrega · envío 2–5 días hábiles ·   │
        │  cambios y devoluciones · horario y NÚNICO número │
        │  visible · reseñas · páginas legales en el footer │
        └──────────────────────┬────────────────────────────┘
                               ▼
   ┌──────────────────────────────────────────────────────────┐
   │  ACCIÓN ÚNICA: "Pedir por WhatsApp"                      │
   │  - Botón principal grande en la ficha                    │
   │  - Barra fija inferior en celular (sticky CTA bar)       │
   │  - Abre https://wa.me/57NUMERO?text=… (encodeURIComponent)│
   │  - target="_blank" rel="noopener"                        │
   │  - Mensaje prellenado: nombre, sku, precio, cantidad,    │
   │    variante elegida, enlace de la ficha (con UTM)        │
   │  ► EVENTO MEDIBLE: whatsapp_click {id, categoria, origen}│
   └───────────────┬──────────────────────────────────────────┘
                   ▼
        ┌───────────────────────────────────────────┐
        │  WHATSAPP BUSINESS (cierre humano)        │
        │  Asesor confirma: nombre, ciudad,        │
        │  dirección, valor del envío, medio de     │
        │  pago (contra entrega cuando aplique)     │
        └──────────────────┬────────────────────────┘
                           ▼
        ┌───────────────────────────────────────────┐
        │  DESPACHO Y SEGUIMIENTO                   │
        │  Transportadora → entrega → confirmación  │
        │  → reseña → recompra / referidos          │
        └───────────────────────────────────────────┘
```

## 2. Flujo alterno: producto agotado (`stock: false`)

```
Ficha con stock:false
   → El botón "Pedir por WhatsApp" se REEMPLAZA por "Avísame cuando llegue"
   → Abre el mismo wa.me con mensaje distinto (solicitud de aviso, no pedido):
     "Hola Nelstore, este producto está agotado. Avísenme cuando vuelva:
      • Producto / Referencia / Enlace"
   ► EVENTO: whatsapp_click con intent:"restock_alert"
```

## 3. Flujos secundarios (todas las páginas)

```
Botón flotante WhatsApp (globales) ──► consulta GENERAL (mensaje distinto al de compra)
Encabezado / buscador  ──► autocompletado normalizado (sin tildes/mayúsculas) ──► /busqueda
Footer ──► envíos · cambios · devoluciones · privacidad · términos · contacto · FAQ
404    ──► búsqueda sugerida + categorías + CTA WhatsApp
```

## 4. Qué se mide (métrica principal del negocio)

**Cada clic a WhatsApp es la conversión del sitio.** Ver lista completa de eventos
en `docs/08-eventos-medicion.md`. KPI primario:
`clics WhatsApp / sesiones` por producto y por canal (UTM).
Si hay muchos clics pero pocos pedidos confirmados → el problema está en precio,
confianza o atención en el chat, **no en el diseño**.

## 5. Reglas de posicionamiento del CTA

| Ubicación | Comportamiento |
|---|---|
| Tarjeta de producto (grid) | Secundaria: “Ver producto” (link a ficha). La compra NO se inicia desde la card. |
| Ficha de producto (desktop) | Botón principal grande junto al bloque de precio/compra (columna derecha). |
| Ficha de producto (celular) | **Barra fija inferior** siempre visible: precio + “Pedir por WhatsApp” (≥48 px alto). |
| Todas las páginas | Botón flotante de WhatsApp con mensaje de consulta general. |
| Sin JavaScript | El botón es un `<a href>` real con el mensaje ya codificado en el HTML servido → sigue funcionando. |
