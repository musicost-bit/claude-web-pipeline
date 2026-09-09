---
name: nextjs-architecture-master
description: Skill maestro que define la arquitectura completa de un proyecto Next.js App Router bajo JAMstack — amarra app/, components/, lib/, types/ y data/ en un solo modelo mental, con dirección de dependencias, flujo de datos, rendering strategy, checklist técnico para agregar features nuevas, y criterios de decisión (SSG vs ISR vs dinámico, cuándo migrar de JSON a CMS). Usar este skill como referencia arquitectónica de alto nivel siempre que se planifique una feature nueva, se revise la coherencia estructural del proyecto completo, se tome una decisión que cruza varias capas, o se necesite explicar/justificar el diseño técnico del sitio. Consultar los skills de capa individual (nextjs-app-structure, nextjs-components-structure, nextjs-lib-structure, nextjs-types-structure, nextjs-data-structure) para el detalle de cada uno.
user-invocable: false
---

# Arquitectura maestra: Next.js App Router + JAMstack

Modelo arquitectónico de referencia para sitios de catálogo/vitrina construidos con Next.js (App Router), Tailwind CSS y datos estáticos o semi-estáticos, sin backend propio. Este skill amarra las cinco capas individuales en un único sistema coherente y define las reglas que cruzan capas.

## Clasificación arquitectónica formal

- **Patrón de infraestructura:** JAMstack (JavaScript + APIs + Markup pre-renderizado). Sin servidor de aplicación propio corriendo 24/7; el sitio se compila a artefactos estáticos servidos desde CDN (Vercel Edge Network).
- **Estrategia de rendering:** SSG (Static Site Generation) como default, con superficie de escape a ISR (Incremental Static Regeneration) si se conecta una fuente de datos externa con `revalidate`.
- **Patrón de routing:** File-based routing (convención de Next.js App Router) — la topología de `app/` define 1:1 la topología de URLs públicas.
- **Patrón de organización de código:** Colocation por dominio/feature, no MVC clásico. La separación real no es "vista/controlador/modelo" sino **rutas (`app/`) / presentación (`components/`) / acceso a datos (`lib/`) / contratos (`types/`) / fuente de datos (`data/`)**.
- **Paradigma de componentes:** React Server Components (RSC) por defecto, con Client Components como excepción explícita (`'use client'`).

## Diagrama de dependencias (regla de dirección única)

```
        ┌─────────────┐
        │   types/     │  ← contratos, sin dependencias hacia otras capas
        └──────┬──────┘
               │ (todas las capas importan de aquí)
   ┌───────────┼────────────┬──────────────┐
   │           │            │              │
┌──▼───┐   ┌───▼───┐   ┌────▼────┐   ┌─────▼─────┐
│ data/ │  │  lib/  │   │components/│  │   app/    │
└──────┘   └───┬───┘   └────┬────┘   └─────┬─────┘
               │             │              │
               └─────────────┴──────────────┘
                  app/ consume lib/ Y components/
                  components/ NUNCA consume lib/ directo
                  (recibe datos ya resueltos vía props)
```

**Regla de dirección única — no negociable:**
1. `types/` no depende de nada — es la base.
2. `data/` no depende de nada (son datos planos, si aplica), pero su forma está gobernada por `types/`.
3. `lib/` depende de `types/` y `data/` (o de una fuente externa). Nunca depende de `components/`.
4. `components/` depende de `types/` (para tipar props). Nunca depende de `lib/` directamente.
5. `app/` es el único orquestador: depende de `lib/` (para obtener datos) y de `components/` (para renderizarlos).

Si en algún punto del código se detecta una flecha en sentido inverso, es una violación arquitectónica que hay que corregir antes de continuar.

## Flujo de datos de una request (build time, caso SSG)

```
1. Build time: Next.js recorre app/**/page.tsx
2. page.tsx con generateStaticParams() → lib/ resuelve qué slugs existen
3. lib/ lee la fuente de datos (JSON local o fuente externa)
4. lib/ devuelve datos tipados según types/
5. page.tsx pasa esos datos como props a components/
6. components/ (Server Components) renderizan HTML en build time
7. Next.js genera archivo estático .html por cada ruta
8. Deploy a Vercel CDN → se sirve sin cómputo por request
```

Para Client Components (`'use client'`), el flujo cambia en el paso 6-8: el componente se serializa, se envía su JS al navegador, y se hidrata en cliente tras cargar el HTML estático.

## Decisión técnica: estrategia de rendering por tipo de página

| Tipo de página | Estrategia | Mecanismo |
|---|---|---|
| Home, Sobre nosotros, Contacto | SSG puro | Sin `generateStaticParams`, contenido fijo en build |
| Listado de catálogo | SSG o ISR | Depende de si la fuente de datos es local o externa — ver nextjs-data-structure |
| Detalle de producto | SSG con rutas pre-generadas | `generateStaticParams()` enumera todos los slugs/IDs |
| Fuente externa que cambia sin control del desarrollador | ISR | `fetch(..., { next: { revalidate: N } })` dentro de `lib/` |
| Data en tiempo real por request | Dinámico (SSR) | `export const dynamic = 'force-dynamic'` — evitar salvo necesidad real, rompe el modelo JAMstack |

**Regla de decisión:** por defecto siempre SSG. Subir a ISR cuando haya una fuente de datos externa que cambie sin control del desarrollador. Solo considerar SSR dinámico si hay contenido que literalmente cambia por request.

## Checklist técnico para agregar una feature nueva

1. **`types/`** — ¿existe ya el tipo que necesito, o hay que definir uno nuevo/extender uno existente? Definir el contrato primero.
2. **`data/`** (si aplica, fuente local) — agregar/editar el JSON con la forma exacta definida en el paso 1.
3. **`lib/`** — escribir la función de acceso a datos que devuelve datos con la forma de `types/`. Función pura, tipada, sin JSX.
4. **`components/`** — construir el/los componente(s) de presentación que reciben esos datos vía props. Decidir explícitamente Server vs Client Component.
5. **`app/`** — crear o modificar la ruta que orquesta: llama a `lib/`, pasa el resultado a `components/`. Agregar `metadata` para SEO. Si es ruta dinámica nueva, agregar `generateStaticParams`.
6. **Validación cruzada** — confirmar que ningún paso rompió la regla de dirección única de dependencias.

Este orden (types → data → lib → components → app) es intencional: cada capa depende solo de las anteriores.

## Cross-cutting concerns

### SEO
- Cada `page.tsx` exporta `metadata` — ver `nextjs-app-structure`.
- Los identificadores/slugs en `data/`/`types/` deben ser SEO-friendly — ver `nextjs-data-structure`.
- SSG garantiza HTML completo en el primer response, crítico para crawlers.

### Performance
- Minimizar superficie de `'use client'`.
- Usar `next/image` en vez de `<img>` plano.
- `generateStaticParams` + SSG elimina cómputo por request.

### Type-safety end-to-end
- El tipo definido en `types/` es el contrato que atraviesa las cinco capas.
- Cualquier `any` en cualquier capa es una fuga en esta cadena de garantías.

## Matriz de decisión rápida: "¿en qué capa va esto?"

| Necesito... | Va en |
|---|---|
| Definir la URL de una página nueva | `app/` |
| Compartir navbar/footer entre páginas | `app/layout.tsx` |
| Un componente visual reutilizable | `components/` |
| Un componente con estado/interactividad | `components/` + `'use client'` |
| Leer/transformar datos de una fuente | `lib/` |
| Formatear un precio, generar un slug | `lib/utils.ts` |
| Declarar la forma de una entidad | `types/` |
| El contenido real (productos, categorías) | `data/` (si la fuente es local) |
| Configuración repetida (teléfono, constantes) | `lib/` (archivo de config) |

## Don'ts arquitectónicos (nivel sistema, no por capa)

- ❌ No saltarse capas — ej. `app/page.tsx` leyendo la fuente de datos directo, sin pasar por `lib/`.
- ❌ No introducir estado global (Context/Redux/Zustand) para necesidades simples — sobre-ingeniería.
- ❌ No mezclar estrategias de rendering sin justificación documentada.
- ❌ No dejar que decisiones de UI (`components/`) determinen la forma de los datos (`types/`).
- ❌ No duplicar lógica de negocio entre `lib/` y `components/`.

## Cómo evoluciona esta arquitectura (rutas de crecimiento previstas)

| Si el proyecto necesita... | Cambio arquitectónico | Capas afectadas |
|---|---|---|
| Edición de catálogo por alguien no técnico | JSON → Google Sheets/CMS | Solo `lib/` (ver `nextjs-data-structure`) |
| Checkout y pagos en línea | Introducir backend/commerce engine | Nueva capa de API — `app/` pasa a consumir una API externa vía `lib/` |
| Actualización de contenido sin rebuild | SSG → ISR | Solo `lib/` (agregar `revalidate`) |
| Multi-idioma | i18n routing de Next.js | `app/` (estructura de rutas) + `data/`/`types/` (campos por idioma) |
| Búsqueda de productos | Filtrado en memoria si el catálogo es chico; servicio externo si crece | `components/` (UI) + `lib/` (integración) |

La propiedad clave de esta arquitectura es que la mayoría de estos crecimientos futuros **no tocan `app/` ni `components/`** — quedan contenidos en `lib/` y `types/`.

## Skills de capa (detalle de implementación)

- `nextjs-app-structure` — rutas, layouts, metadata, Server/Client Components en `app/`
- `nextjs-components-structure` — componentes de presentación, composición, `'use client'`
- `nextjs-lib-structure` — acceso a datos, funciones puras, punto único de cambio
- `nextjs-types-structure` — contratos/DTOs, `interface` vs `type`
- `nextjs-data-structure` — fuente de datos, forma del contenido, criterios de migración
