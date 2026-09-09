---
name: nextjs-lib-structure
description: Mejores prácticas, recomendaciones, limitantes y errores comunes para estructurar el directorio lib/ en proyectos Next.js (App Router) — la capa de acceso a datos y utilidades, equivalente a repository/service layer en backend tradicional (Java/.NET). Usar este skill siempre que se cree, revise o refactorice lib/, se implemente una función que lea datos (JSON, Google Sheets, CMS, API externa), se agreguen helpers/utils, o se decida dónde debe vivir una pieza de lógica que no es ni ruta ni componente visual. Complementa a nextjs-app-structure y nextjs-components-structure.
user-invocable: false
---

# Estructura de `lib/` en Next.js (App Router)

Guía de referencia para la capa de acceso a datos y utilidades en proyectos Next.js, pensada para que alguien con background backend (Java/.NET/etc.) reconozca el patrón: `lib/` cumple el rol de **repository/service layer**.

## Qué va dentro de `lib/`

Todo lo que **no es ruta** (`app/`) y **no es presentación** (`components/`):

- Funciones que obtienen datos (de JSON local, Google Sheets, un CMS, una API externa).
- Lógica de negocio simple (formatear precios, generar slugs, validar datos).
- Utilidades genéricas reutilizables en todo el proyecto.

`lib/` es la **única capa que sabe de dónde vienen los datos**. Ni `app/` ni `components/` deberían saber si los productos vienen de un archivo JSON, una hoja de Google Sheets o un CMS — solo llaman a una función de `lib/` y reciben el dato ya tipado.

## Estructura recomendada

```
lib/
├── products.ts          # Acceso a datos de productos (repository)
├── whatsapp.ts           # Helpers para generar links de WhatsApp
└── utils.ts               # Helpers genéricos (formatear precio, fechas, etc.)
```

Si el proyecto crece, separar por dominio:

```
lib/
├── products/
│   ├── get-products.ts
│   └── get-product-by-slug.ts
├── whatsapp.ts
└── utils.ts
```

## Mejores prácticas

### 1. Una función, una responsabilidad clara (patrón repository)

```ts
// lib/products.ts
import { Product } from '@/types/product'

export async function getAllProducts(): Promise<Product[]> {
  // implementación según la fuente de datos real del proyecto
}

export async function getProductBySlug(slug: string): Promise<Product | undefined> {
  const products = await getAllProducts()
  return products.find((p) => p.slug === slug)
}
```

Esto es exactamente el rol de un `ProductRepository` en Spring/EF Core, solo que sin clases — funciones puras que devuelven datos tipados.

### 2. Aislar la fuente de datos — punto único de cambio

Si hoy los datos vienen de un JSON y mañana pasan a Google Sheets o un CMS, **solo se toca `lib/products.ts`**. Nada en `app/` ni `components/` debería cambiar, porque ellos solo conocen la firma de la función, no su implementación.

```ts
// lib/products.ts — ejemplo con fuente externa (Google Sheets)
export async function getAllProducts(): Promise<Product[]> {
  const res = await fetch(SHEETS_API_URL, { next: { revalidate: 3600 } })
  const data = await res.json()
  return mapSheetRowsToProducts(data)
}
```

### 3. Funciones puras cuando sea posible

```ts
// lib/utils.ts
export function formatPrice(amount: number, currency: string = 'PEN'): string {
  return new Intl.NumberFormat('es-PE', { style: 'currency', currency }).format(amount)
}
```

### 4. Separar "acceso a datos" de "utilidades genéricas"

`products.ts` (sabe de productos) es distinto de `utils.ts` (no sabe nada del dominio).

### 5. Tipar siempre la entrada y salida

```ts
import { Product } from '@/types/product'

export async function getFeaturedProducts(limit: number = 4): Promise<Product[]> {
  const products = await getAllProducts()
  return products.slice(0, limit)
}
```

### 6. Centralizar constantes de configuración relacionadas

```ts
// lib/whatsapp.ts
const WHATSAPP_PHONE = '51XXXXXXXXX'

export function buildWhatsAppLink(productName: string): string {
  const message = encodeURIComponent(`Hola, me interesa el producto: ${productName}`)
  return `https://wa.me/${WHATSAPP_PHONE}?text=${message}`
}
```

## Limitantes a tener en cuenta

- **Funciones síncronas vs. `async`:** si la fuente de datos es externa (Sheets, CMS, API), las funciones de `lib/` son `async`/`Promise` — esto obliga a los `page.tsx` que las consumen a usar `await`.
- **`lib/` no debe importar de `components/`.** La dirección de dependencia es: `app/` → `lib/` y `app/` → `components/`, pero nunca `lib/` → `components/`.
- **Revalidación de datos (ISR):** si se usa `fetch` con `next: { revalidate: N }`, ese comportamiento de cacheo queda atado a esa función — documentarlo bien.

## Don'ts (errores comunes)

- ❌ No hacer fetching de datos directo dentro de `page.tsx` sin pasar por `lib/`.
- ❌ No poner JSX ni nada de React dentro de `lib/`.
- ❌ No hardcodear valores de configuración repetidos en varios componentes — centralizarlos en `lib/`.
- ❌ No mezclar transformación de datos con validación de formularios y utilidades genéricas en un solo archivo gigante.
- ❌ No omitir tipos de retorno explícitos en funciones exportadas.
- ❌ No duplicar lógica de acceso a datos en distintos `page.tsx`.

## Skills relacionados

- `nextjs-app-structure` — mejores prácticas para `app/` (rutas y layouts, consume `lib/`)
- `nextjs-components-structure` — mejores prácticas para `components/` (presentación, recibe datos de `lib/` vía props)
