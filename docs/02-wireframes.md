# NELSTORE — Wireframes en texto (mobile-first)

Convenciones: `[ ]` botón · `( )` campo de formulario · `▾` desplegable · `≡` menú hamburguesa
Los wireframes se describen primero en **móvil (360 px)**; las notas de **desktop (≥1024 px)** van al final de cada página.

---

## 1. INICIO (`index.html`)

```
┌──────────────────────────────┐
│ ▲ ENVÍO A TODA COLOMBIA · PAGO CONTRA ENTREGA ·       │ ← barra superior de anuncios
│   ATENCIÓN POR WHATSAPP     (marquee/rotativo, opc.)  │   (fondo azul oscuro, texto blanco, 1 línea)
├──────────────────────────────┤
│ ≡  NELSTORE      [ 🔍 Buscar productos...   ]  [WA]   │ ← encabezado sticky
│    ▾ Categorías                                        │   (logo + buscador grande + WA + menú)
├──────────────────────────────┤
│ ┌──────────────────────────┐ │
│ │  BANNER CARRUSEL         │ │  ← 2-3 slides: imagen + titular corto
│ │  "Recupera tu cuerpo"    │ │     + botón "Ver productos"
│ │        ● ○ ○             │ │     lazy load, altura fija (evita CLS)
│ └──────────────────────────┘ │
├──────────────────────────────┤
│ 🚚 Envíos    💵 Contra   💬 WhatsApp  🔒 Compra       │ ← íconos de confianza
│ a toda CO     entrega       24/7 hab.                 │   (grid 2x2 móvil / 4 desktop)
├──────────────────────────────┤
│ CATEGORÍAS                   │
│ ┌────────┐ ┌────────┐        │
│ │  🖼    │ │  🖼    │        │ ← tarjetas de categoría: ícono/foto + nombre
│ │Masajea-│ │Soporte │        │   ("Más vendidos" badge opcional)
│ │res     │ │articult│        │
│ └────────┘ └────────┘        │
│ ┌────────┐ ┌────────┐        │
│ │Fitness │ │Cuidado │        │
│ └────────┘ └────────┘        │
├──────────────────────────────┤
│ ⏱ OFERTAS DEL DÍA    Ver todas ▸│
│ ┌────────┐ ┌────────┐ ┌─···   │ ← carrusel horizontal con scroll-snap (móvil)
│ │ [IMG]  │ │ [IMG]  │         │
│ │ -25%   │ │ -30%   │         │ ← badge descuento sobre la imagen
│ │ Nombre │ │ Nombre │         │   nombre máx. 2 líneas (…)
│ │ $49.900│ │ $32.000│         │   precio actual destacado (azul/negro)
│ │$65.900 │ │$45.900 │         │   precio anterior tachado, gris
│ │★★★★☆   │ │★★★★★   │         │   calificación + nº reseñas
│ └────────┘ └────────┘         │
├──────────────────────────────┤
│ 🔥 MÁS VENDIDOS          ▸   │ ← misma tarjeta de producto, grid 2 col móvil
├──────────────────────────────┤
│ ✨ RECIÉN LLEGADOS        ▸   │ ← badge "Nuevo" verde/turquesa
├──────────────────────────────┤
│ ¿POR QUÉ NELSTORE?           │
│ ✔ Especialistas en bienestar │ ← bloque de beneficios (lista o 3 tarjetas)
│ ✔ Selección verificada       │
│ ✔ Atención real por WhatsApp │
│ ✔ Pago contra entrega        │
├──────────────────────────────┤
│ LO QUE DICEN NUESTROS        │ ← testimonios: carrusel o grid, estrellas +
│ CLIENTES                     │   nombre + ciudad + texto corto (sin claims médicos)
│ ★★★★★ "Llegó rápido y ..."  │
│  — Camila R., Bogotá         │
├──────────────────────────────┤
│ PREGUNTAS FRECUENTES         │ ← 4-5 acordeones breves + enlace "Ver todas ▸"
│ ▸ ¿Cómo hago mi pedido?      │
│ ▸ ¿Cuánto tarda el envío?    │
├──────────────────────────────┤
│ FOOTER (ver componente)      │
└──────────────────────────────┘
        ◉ WhatsApp flotante (abajo-derecha, siempre visible)
```

**Desktop:** banner full-width · confianza en 4 columnas · categorías 4-col ·
secciones de productos en grid 4-col · testimonios 3-col.

---

## 2. CATEGORÍA (`categorias/plantilla-categoria.html` — reutilizable)

```
┌──────────────────────────────┐
│ Header + buscador (compartido)│
├──────────────────────────────┤
│ Inicio ▸ Masajeadores         │ ← breadcrumbs
├──────────────────────────────┤
│ MASAJEADORES Y RECUPERACIÓN  │ ← H1 + descripción SEO corta (2 líneas)
│ "Equipos para el cuidado      │
│  post-entrenamiento..."       │
├──────────────────────────────┤
│ [ ☰ Filtros ]  [ Ordenar: Más relevantes ▾ ] │ ← móvil: botones que abren panel
│                                              │   desktop: sidebar fijo izq. (240px)
│ ┌ DESKTOP: SIDEBAR ─────────┐│
│ │ Precio: ( )rango sliders  ││
│ │ Calificación: ★★★★☆ y más││ ← panel lateral filtros (móvil = drawer desde abajo)
│ │ Disponibilidad: ✔ En stock││
│ │ Ofertas: ☑ Solo con dto.  ││
│ │ [ Aplicar ] [ Limpiar ]   ││
│ └───────────────────────────┘│
│ 24 resultados                  │
├──────────────────────────────┤
│ ┌────────┐ ┌────────┐         │ ← cuadrícula: 2 col móvil / 3 tablet / 4-5 desktop
│ │ Tarjeta│ │ Tarjeta│         │   (componente ProductCard, ver lista)
│ │  prod. │ │  prod. │         │
│ └────────┘ └────────┘         │
│ ┌────────┐ ┌────────┐         │
│ │  ...   │ │  ...   │         │
├──────────────────────────────┤
│ [ VER MÁS PRODUCTOS ]          │ ← paginación infinita suave o botón "ver más"
│                                │   (fallback: numerada ‹ 1 2 3 ›)
├──────────────────────────────┤
│ Footer (compartido)            │
└──────────────────────────────┘
```

*Estado vacío:* "No encontramos productos con esos filtros" + botón limpiar + CTA WhatsApp.

---

## 3. RESULTADOS DE BÚSQUEDA (`busqueda.html`)

```
┌──────────────────────────────┐
│ Header (buscador enfocado)     │
├──────────────────────────────┤
│ Inicio ▸ Búsqueda              │
│ Resultados para: "rodillera"  │ ← H1 con el término (decodeado, normalizado)
│ 8 productos encontrados        │
├──────────────────────────────┤
│ Misma barra de filtros/orden   │
│ Misma cuadrícula ProductCard   │ ← búsqueda insensible a tildes/mayúsculas
│                                │   sobre: nombre + descripción + categoría + etiquetas
├──────────────────────────────┤
│ 0 resultados → sugerencias:    │
│ "Quizás buscabas: rodilleras"  │
│ [Más vendidos] [Preguntar por WhatsApp] │
└──────────────────────────────┘
```

---

## 4. FICHA DE PRODUCTO (`producto/[slug].html` — reutilizable)

```
┌──────────────────────────────┐
│ Header compartido              │
├──────────────────────────────┤
│ Inicio ▸ Fitness ▸ Shaker ...  │ ← breadcrumbs con categoría real del JSON
├──────────────────────────────┤
│ ┌ GALERÍA ─────────────────┐  │
│ │                          │  │ ← imagen principal cuadrada, zoom onhover/tap
│ │      [ IMG 1:1 ]    🔍   │  │   (lupa → modal/fullscreen en móvil)
│ │  ○ ○ ● ○  (miniaturas)   │  │   miniaturas debajo (horizontal scrolleable)
│ └──────────────────────────┘  │
├──────────────────────────────┤
│ SHAKER PROTEICO 600 ML         │ ← H1 (nombre)
│ Ref: NEL-SHK-001               │ ← referencia/SKU visible (va en el msg WhatsApp)
│ ★★★★★ 4.6 (32 reseñas)         │ ← rating + enlace a pestaña reseñas
│                                │
│ $32.900   −20%  ~~$41.000~~    │ ← precio destacado grande, % dto badge, anterior tachado
│ ✔ En stock · Envío desde $9.900│ ← disponibilidad + info envío
│                                │
│ Incluye: medida graduada,      │ ← descripción corta (3 bullets de beneficios,
│  tapa antigoteo, libre de BPA  │   SIN afirmaciones médicas)
│                                │
│ ┌──────────────────────────┐  │
│ │  [ 📱 PEDIR POR WHATSAPP ]│  │ ← CTA PRIMARIO, ancho completo, verde WA
│ └──────────────────────────┘  │   prellena: "Hola, quiero pedir el producto
│ 💵 Pago contra entrega disponible│   {nombre} (Ref: {sku}) por ${precio}"
│ 🚚 Entrega estimada 2-5 días   │ ← caja de info envío/confianza bajo el CTA
│ 🔒 Compra asistida por WhatsApp│
├──────────────────────────────┤
│ [Descripción][Características] │ ← pestañas (acordeón en móvil):
│ [Modo de uso][Envíos y devol.] │   descripción completa / tabla specs /
│                                │   modo de uso e imágenes / política
├──────────────────────────────┤
│ ℹ INVIMA: Reg. Sanitario #...  │ ← SOLO si el campo invima existe en el JSON
├──────────────────────────────┤
│ PRODUCTOS RELACIONADOS         │ ← misma categoría u ofertas, carrusel/grid 2-4
│ [Card][Card][Card][Card]       │
├──────────────────────────────┤
│ Footer                         │
└──────────────────────────────┘
```

---

## 5. OFERTAS (`ofertas.html`)

```
┌──────────────────────────────┐
│ Header                         │
├──────────────────────────────┤
│ 🔥 OFERTAS                     │ ← H1 + contador regresivo opcional
│ ⏳ Termina en 02d : 14h : 33m  │   (configurable, no fake countdown permanente)
├──────────────────────────────┤
│ [ ☰ Filtros ] [ Ordenar ▾ ]    │ ← mismos controles de la plantilla categoría
├──────────────────────────────┤
│ Grid de tarjetas SIEMPRE con   │ ← solo productos oferta=true y vigente
│ badge −XX% y precio anterior   │   (reutiliza la lógica de categoria.html)
│ [VER MÁS]                      │
├──────────────────────────────┤
│ Footer                         │
└──────────────────────────────┘
```

---

## 6–10. Páginas informativas (resumen de wireframe)

| Página | Bloques (en orden) |
|---|---|
| **Sobre nosotros** | Hero con foto real → Nuestra historia → Misión/Visión → Valores (3-4 tarjetas) → Por qué especializarnos en bienestar → CTA WhatsApp |
| **Contacto** | H1 → Canales: [WhatsApp] (tarjeta grande), correo, horario (L-S 8:00-20:00) → Formulario `(Nombre)(Teléfono)(Correo)(Asunto ▾)(Mensaje)[Enviar]` → nota de tratamiento de datos + checkbox consentimiento (Ley 1581) |
| **FAQ** | H1 → buscador de preguntas (opcional) → acordeones agrupados: Pedidos · Envíos · Pagos · Devoluciones · Productos → CTA "¿otra duda? WhatsApp" |
| **Legales (×4)** | Layout estático simple: H1 + fecha de actualización + secciones numeradas + anclas laterales en desktop |
| **404** | Ícono/ilustración + "Página no encontrada" + buscador grande + chips de categorías + grid "Más vendidos" + botón Volver al inicio |
