# NELSTORE — Mapa del sitio (Etapa 1: Estructura)

> Tienda de un solo nicho: salud, bienestar y cuidado personal (Colombia).
> Cierre de venta por WhatsApp. Navegación tipo marketplace, identidad limpia y confiable.

## Árbol de páginas

```
nelstore.co/
│
├── index.html .................... INICIO (home)
│
├── categorias/ ................... PLANTILLA ÚNICA de categoría
│   ├── masajeadores-recuperacion.html      ?cat=masajeadores
│   ├── soporte-articulaciones.html         ?cat=soporte
│   ├── fitness-deporte.html                ?cat=fitness
│   ├── cuidado-personal.html               ?cat=cuidado
│   └── (nueva categoria = 1 entrada en categories.json, sin rediseño)
│       ej.: suplementos.html, cuidado-piel.html
│
├── producto/ ..................... PLANTILLA ÚNICA de ficha de producto
│   └── [slug].html  .............. lee ?id= o el slug → products.json
│
├── ofertas.html .................. OFERTAS (página independiente, contador opcional)
├── busqueda.html ................. RESULTADOS DE BÚSQUEDA (misma cuadrícula que categoría)
│
├── sobre-nosotros.html ............ Historia, misión, valores, por qué el nicho
├── contacto.html ................. Formulario + WhatsApp + correo + horario
├── faq.html ...................... Preguntas frecuentes (acordeón)
│
├── legales/
│   ├── envios-y-entregas.html
│   ├── cambios-y-devoluciones.html
│   ├── terminos-y-condiciones.html
│   └── politica-privacidad.html .. incl. tratamiento de datos (Ley 1581/2012, Colombia)
│
└── 404.html ...................... Página 404 amigable (buscador + categorías + más vendidos)
```

## Navegación principal (encabezado)

| Nivel 1 | Contenido |
|---|---|
| Logo | → inicio |
| Buscador grande | autocompletado desde products.json (nombre, categoría, etiquetas) |
| Menú "Categorías" | dropdown / panel con las 4 categorías + Ofertas |
| Ofertas | enlace destacado (color acento) |
| Botón WhatsApp | flotante en móvil, inline en desktop |

**Footer:** enlaces legales, contacto, horario, categorías, redes, métodos de confianza
(envío a toda Colombia, pago contra entrega, atención por WhatsApp).

## Jerarquía de categorías (extensible)

```
categories.json  (fuente única de verdad para el menú)
├── masajeadores   "Masajeadores y recuperación"
├── soporte        "Soporte y articulaciones"
├── fitness        "Fitness y deporte"
├── cuidado        "Cuidado personal"
└── ofertas        (vista transversal: productos con oferta=true, no es categoría de catálogo)
```

Regla de extensibilidad: agregar una categoría nueva = 1 objeto en `categories.json`
+ su página física generada desde la misma plantilla `categorias/*.html`.
No se toca CSS, ni encabezado, ni lógica de filtrado.

## Flujo de compra (sin checkout propio)

```
Inicio / Categoría / Búsqueda / Ofertas
        └─► Ficha de producto
              └─► [Pedir por WhatsApp]  → wa.me/57XXXXXXXXXX?text=
                    mensaje prellenado: nombre + referencia (SKU) + precio
                    └─► Confirmación por chat → pago contra entrega / transferencia
```
