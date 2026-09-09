---
name: nextjs-app-structure
description: Mejores prácticas, recomendaciones, limitantes y errores comunes para estructurar el directorio app/ en proyectos Next.js (App Router) siguiendo arquitectura JAMstack con organización por dominio/feature. Usar este skill siempre que se cree, revise o refactorice la carpeta app/ de un proyecto Next.js, se definan rutas nuevas, layouts, páginas, o se tomen decisiones sobre Server vs Client Components. Aplica especialmente a sitios de catálogo/vitrina (ecommerce sin checkout, landing pages, sitios de contenido) construidos con Next.js + Tailwind.
user-invocable: false
---

# Estructura de `app/` en Next.js (App Router)

Guía de referencia para mantener homogénea la forma en que se construye el directorio `app/` en proyectos Next.js con App Router, bajo arquitectura JAMstack (sitios estáticos/pre-renderizados, sin backend propio corriendo 24/7).

## Contexto de arquitectura

- **Patrón general:** JAMstack — JavaScript + APIs + Markup pre-renderizado. Contenido servido como archivos estáticos desde CDN (ej. Vercel), sin servidor backend propio.
- **Routing:** File-based routing — la estructura de carpetas dentro de `app/` ES la estructura de URLs del sitio.
- **Organización de código:** Por dominio/feature (colocation), no por tipo técnico puro tipo MVC clásico.

## Qué va dentro de `app/`

Solo tres responsabilidades: **rutas, layouts y configuración de rendering**. Nada de lógica de negocio, nada de fetching complejo inline, nada de estilos sueltos fuera del layout.

## Archivos especiales

| Archivo | Función | Obligatorio |
|---|---|---|
| `page.tsx` | Define una ruta pública/visitable | Sí, para que la carpeta sea una URL |
| `layout.tsx` | Envoltorio compartido (navbar, footer, providers) | Solo en `app/` raíz; opcional en subcarpetas |
| `loading.tsx` | UI de carga automática (Suspense boundary) | No, pero recomendado en rutas con fetch |
| `error.tsx` | Manejo de errores de esa ruta hacia abajo | No, pero recomendado |
| `not-found.tsx` | Página 404 personalizada | No |
| `route.ts` | API endpoint | No, salvo que se exponga una API pública |

**Regla de oro:** una carpeta sin `page.tsx` NO es una ruta navegable — solo sirve para organizar. Las carpetas con prefijo `_` (ej. `_components/`) se excluyen automáticamente del routing.

## Mejores prácticas

### 1. `layout.tsx` raíz mínimo y estable

Solo estructura y providers globales (theme, analytics). Sin lógica de negocio.

```tsx
// app/layout.tsx
export default function RootLayout({ children }: { children: React.ReactNode }) {
  return (
    <html lang="es">
      <body>
        <Navbar />
        {children}
        <Footer />
      </body>
    </html>
  )
}
```

### 2. `page.tsx` delgado — orquesta, no implementa

Lee datos vía `lib/` y los pasa como props a componentes. Nunca JSX de cientos de líneas directo en la página.

```tsx
// app/catalogo/page.tsx
import { getAllProducts } from '@/lib/products'
import { ProductGrid } from '@/components/product/ProductGrid'

export default function CatalogoPage() {
  const products = getAllProducts()
  return <ProductGrid products={products} />
}
```

### 3. Carpetas con guion bajo para "no-rutas"

Para componentes específicos de una sola ruta (no reutilizables en el resto del sitio):

```
app/catalogo/
├── page.tsx
└── _components/
    └── CatalogFilters.tsx   (solo se usa aquí, no en todo el sitio)
```

### 4. Metadata en cada `page.tsx` (SEO)

```tsx
export const metadata = {
  title: 'Catálogo | Mi Tienda',
  description: 'Vitrina de productos...',
}
```

Sin esto se pierde el principal beneficio de SSG para SEO.

### 5. `generateStaticParams` en toda ruta dinámica con datos fijos

```tsx
// app/catalogo/[slug]/page.tsx
export async function generateStaticParams() {
  return getAllProducts().map(p => ({ slug: p.slug }))
}
```

Obligatorio cuando el catálogo no cambia por request — genera páginas estáticas en build time.

### 6. Server Components por defecto

No agregar `'use client'` "por si acaso". Cada `'use client'` aumenta el JS enviado al navegador. Solo usarlo en componentes con `useState`, `useEffect`, `onClick`, etc.

## Limitantes a tener en cuenta

- **`layout.tsx` no se re-renderiza al navegar entre páginas hijas.** Si algo debe cambiar por ruta (ej. título dinámico), va en `page.tsx`/`metadata`, no en el layout.
- **Server Components no pueden usar hooks de React** (`useState`, `useEffect`) ni event handlers (`onClick`). Si un componente necesita eso, debe ser `'use client'`.
- **`generateStaticParams` requiere conocer todos los valores en build time.** Si el catálogo pasa a ser dinámico sin rebuild, se necesita ISR (`revalidate`) o rendering dinámico.
- **Las rutas anidadas heredan layouts.** Un `layout.tsx` en `app/catalogo/` se aplica a todo lo de adentro, incluyendo `[slug]/`. Cuidado con duplicar navbar/footer sin querer.

## Don'ts (errores comunes)

- ❌ No poner fetching de datos ni lógica de negocio directo en componentes de `components/`. El fetch va en `page.tsx` (Server Component) o en `lib/`, y se pasa como props.
- ❌ No usar `'use client'` en el layout raíz — mata la ventaja de Server Components para todo el árbol.
- ❌ No mezclar rutas de páginas con rutas de API sin necesidad real — no crear `route.ts` "por si acaso".
- ❌ No duplicar navbar/footer en cada `page.tsx` — para eso existe `layout.tsx`.
- ❌ No nombrar carpetas de rutas con mayúsculas o camelCase — se convierten literalmente en la URL (`sobre-nosotros`, no `sobreNosotros`).
- ❌ No anidar más de 2-3 niveles de rutas sin necesidad real.
- ❌ No omitir `loading.tsx` en rutas con fetch de datos externos — sin eso, el usuario ve pantalla en blanco mientras carga.

## Ejemplo de estructura completa (referencia)

```
app/
├── layout.tsx
├── page.tsx
├── globals.css
├── catalogo/
│   ├── page.tsx
│   ├── _components/
│   │   └── CatalogFilters.tsx
│   └── [slug]/
│       └── page.tsx
├── sobre-nosotros/
│   └── page.tsx
└── contacto/
    └── page.tsx
```

## Skills relacionados

- `nextjs-components-structure` — mejores prácticas para `components/`
- `nextjs-lib-structure` — mejores prácticas para `lib/` (capa de acceso a datos)
