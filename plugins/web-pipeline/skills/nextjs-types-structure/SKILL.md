---
name: nextjs-types-structure
description: Mejores prácticas, recomendaciones, limitantes y errores comunes para estructurar el directorio types/ en proyectos Next.js con TypeScript — la capa de contratos/modelos, equivalente a DTOs/Entities en backend tradicional (Java/.NET). Usar este skill siempre que se cree, revise o refactorice types/, se defina una nueva interface o type, se decida si un tipo va en types/ o vive local a un archivo, o se modele la forma de datos que vienen de una fuente externa (JSON, Google Sheets, CMS, API). Complementa a nextjs-app-structure, nextjs-components-structure y nextjs-lib-structure.
user-invocable: false
---

# Estructura de `types/` en Next.js + TypeScript

Guía de referencia para la capa de contratos/modelos en proyectos Next.js: `types/` cumple el rol de **DTOs/Entities**, aportando el mismo valor de tipado fuerte que un backend tipado.

## Qué va dentro de `types/`

Definiciones de **forma de datos** (`interface`/`type`) que se comparten entre dos o más capas del proyecto (`lib/`, `components/`, `app/`). Es el contrato que todos respetan.

No van aquí tipos estrictamente internos a un solo componente sin reuso en otro lado — esos pueden vivir localmente en el mismo archivo.

## Estructura recomendada

```
types/
├── product.ts       # Product, ProductCategory, ProductVariant
├── contact.ts         # ContactFormData (si hay formulario)
└── common.ts           # Tipos genéricos compartidos (ej. SEOMetadata)
```

## Mejores prácticas

### 1. `interface` para formas de objeto/entidades, `type` para uniones y alias

```ts
// types/product.ts
export interface Product {
  id: string
  slug: string
  name: string
  description: string
  price: number
  images: string[]
  category: ProductCategory
}

export type ProductCategory = 'lenceria'
```

### 2. Campos opcionales explícitos con `?`, nunca `| undefined` disperso

```ts
// ✅ correcto
interface Product {
  material?: string
}
```

### 3. Un tipo = una responsabilidad; componer en vez de duplicar

```ts
export interface Product {
  id: string
  slug: string
  name: string
  price: number
  images: string[]
}

export interface ProductWithRelated extends Product {
  relatedProducts: Product[]
}
```

### 4. Modelar exactamente la forma real de los datos, no "lo que gustaría que fuera"

Si la fuente de datos tiene inconsistencias, normalizar en `lib/` antes de que llegue tipado limpio al resto del proyecto — mantener el tipo limpio en `types/`.

### 5. Reexportar desde un único punto si el proyecto crece

```ts
// types/index.ts
export * from './product'
export * from './contact'
export * from './common'
```

### 6. Tipos que vienen de una fuente externa van separados de los tipos "de dominio" limpios

```ts
// Forma limpia que usa el resto del proyecto
export interface Product {
  id: string
  slug: string
  name: string
  price: number
}

// Forma cruda tal cual viene de una fuente externa (ej. Google Sheets)
export interface ProductSheetRow {
  ID: string
  Slug: string
  Nombre: string
  Precio: string   // viene como texto
}
```

La función de mapeo vive en `lib/`, no en `types/` — `types/` solo declara las formas, no transforma.

## Limitantes a tener en cuenta

- **TypeScript no valida en runtime.** Si la fuente de datos externa cambia de formato sin avisar, TypeScript no lo detecta en producción. Si la fuente es externa y poco confiable, vale la pena validar en runtime con algo como `zod` en `lib/`.
- **Cambiar un tipo compartido impacta varias capas a la vez.** Si se modifica `Product`, hay que revisar `lib/` y `components/`.

## Don'ts (errores comunes)

- ❌ No usar `any` para "resolver rápido" un tipo que no se quiere modelar.
- ❌ No definir el mismo tipo en dos archivos distintos.
- ❌ No mezclar tipos de UI genéricos dentro de archivos de dominio.
- ❌ No poner lógica ni funciones dentro de `types/`.
- ❌ No dejar props de componentes sin usar los tipos de `types/` cuando corresponde.

## Skills relacionados

- `nextjs-app-structure` — mejores prácticas para `app/` (rutas y layouts)
- `nextjs-components-structure` — mejores prácticas para `components/` (consume tipos vía props)
- `nextjs-lib-structure` — mejores prácticas para `lib/` (devuelve datos con la forma definida en `types/`)
