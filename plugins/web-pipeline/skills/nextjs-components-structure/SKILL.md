---
name: nextjs-components-structure
description: Mejores prácticas, recomendaciones, limitantes y errores comunes para estructurar el directorio components/ en proyectos Next.js (App Router), organizados por dominio/feature con Tailwind CSS. Usar este skill siempre que se cree, revise o refactorice components/, se decida si un componente es Server o Client, se definan props/tipos de componentes, o se evalúe si algo debe ser un componente reutilizable o quedarse local a una ruta. Complementa a nextjs-app-structure.
user-invocable: false
---

# Estructura de `components/` en Next.js (App Router)

Guía de referencia para mantener homogénea la forma en que se construyen componentes en proyectos Next.js con App Router + Tailwind, bajo organización por dominio (colocation) en lugar de MVC clásico.

## Qué va dentro de `components/`

Componentes de **presentación reutilizables** — reciben datos vía props y los renderizan. No hacen fetching de datos propio (salvo excepciones puntuales bien justificadas), no tienen lógica de negocio, no saben de dónde vienen los datos.

## Estructura recomendada

```
components/
├── ui/                    # Componentes genéricos, sin conocimiento del dominio
│   ├── Button.tsx
│   ├── Card.tsx
│   ├── Badge.tsx
│   └── Input.tsx
├── layout/                # Piezas estructurales del sitio
│   ├── Navbar.tsx
│   └── Footer.tsx
└── product/                # Componentes específicos del dominio
    ├── ProductCard.tsx
    ├── ProductGrid.tsx
    ├── ProductDetail.tsx
    └── WhatsAppButton.tsx
```

**Regla de organización:** una carpeta por dominio/feature, no por tipo técnico.

## Mejores prácticas

### 1. Un componente, una responsabilidad

`ProductCard` solo pinta una tarjeta. `ProductGrid` solo arma el grid iterando cards. `WhatsAppButton` solo genera el link de contacto.

```tsx
// components/product/WhatsAppButton.tsx
export function WhatsAppButton({ productName }: { productName: string }) {
  const message = encodeURIComponent(`Hola, me interesa el producto: ${productName}`)
  const phone = '51XXXXXXXXX'
  return (
    <a
      href={`https://wa.me/${phone}?text=${message}`}
      target="_blank"
      rel="noopener noreferrer"
      className="bg-green-500 text-white px-4 py-2 rounded-lg"
    >
      Consultar por WhatsApp
    </a>
  )
}
```

### 2. Tipar props explícitamente con TypeScript

Nunca `any`. Reusar los tipos definidos en `types/` en lugar de redefinir formas de objeto dentro del componente.

```tsx
import { Product } from '@/types/product'

interface ProductCardProps {
  product: Product
}

export function ProductCard({ product }: ProductCardProps) {
  // ...
}
```

### 3. Server Components por defecto, también en `components/`

`ProductCard` (solo muestra datos) no necesita `'use client'`. `WhatsAppButton` tampoco, porque es un link `<a>`, no un botón con estado.

### 4. `'use client'` solo en el componente más pequeño posible

```tsx
// components/product/_client/CategoryFilter.tsx
'use client'
import { useState } from 'react'

export function CategoryFilter({ categories }: { categories: string[] }) {
  const [active, setActive] = useState<string | null>(null)
  // ...
}
```

Así `ProductGrid` alrededor sigue siendo Server Component; solo el filtro puntual baja al cliente.

### 5. `ui/` no debe conocer el dominio

`Button.tsx` en `ui/` no debe importar tipos de `Product`. Si un botón necesita lógica específica de producto, esa lógica va en un componente de `product/` que internamente usa `<Button>` de `ui/`.

### 6. Nomenclatura consistente

- Archivos de componentes: **PascalCase** (`ProductCard.tsx`).
- Carpetas: **kebab-case** o simple (`product/`, no `Product/`).
- Un componente = un archivo con el mismo nombre que el export principal.

### 7. Composición sobre configuración excesiva

Preferir componer componentes pequeños antes que un solo componente gigante con muchas props booleanas controlando variantes.

## Límites y cuándo SÍ hacer fetching dentro de `components/`

- Un componente dentro de `components/` puede ser `async` y hacer fetching solo si es un Server Component y el dato es estrictamente propio de ese componente. Es la excepción, no la norma.
- Nunca hacer fetching dentro de un Client Component directamente en el render.

## Don'ts (errores comunes)

- ❌ No poner lógica de negocio dentro de un componente — eso va en `lib/`.
- ❌ No hacer que `ui/` importe algo de `product/`.
- ❌ No crear un componente "god" con 10+ props controlando variantes.
- ❌ No usar `'use client'` en un contenedor grande solo porque un hijo pequeño lo necesita.
- ❌ No mezclar componentes reutilizables con componentes de un solo uso en la misma carpeta — los de un solo uso van en `_components/` dentro de la ruta que los usa.
- ❌ No duplicar un componente casi idéntico en dos carpetas de dominio.
- ❌ No dejar componentes sin tipar props (`props: any`).

## Ejemplo de estructura completa (referencia)

```
components/
├── ui/
│   ├── Button.tsx
│   ├── Card.tsx
│   └── Badge.tsx
├── layout/
│   ├── Navbar.tsx
│   └── Footer.tsx
└── product/
    ├── ProductCard.tsx
    ├── ProductGrid.tsx
    ├── ProductDetail.tsx
    ├── WhatsAppButton.tsx
    └── _client/
        └── CategoryFilter.tsx
```

## Skills relacionados

- `nextjs-app-structure` — mejores prácticas para `app/` (rutas y layouts)
- `nextjs-lib-structure` — mejores prácticas para `lib/` (capa de acceso a datos, siguiente paso lógico)
