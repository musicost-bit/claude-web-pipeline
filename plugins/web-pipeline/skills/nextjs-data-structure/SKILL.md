---
name: nextjs-data-structure
description: Mejores prácticas, recomendaciones, limitantes y errores comunes para estructurar el directorio data/ en proyectos Next.js que usan archivos JSON locales como fuente de datos (catálogos estáticos, sitios de vitrina sin backend/CMS/base de datos propia). Usar este skill siempre que se cree, revise o refactorice data/, se agregue o edite un archivo JSON de contenido, se decida la forma de un producto/entidad en JSON, o se evalúe cuándo migrar de JSON local a una fuente externa (Google Sheets, CMS). Complementa a nextjs-lib-structure y nextjs-types-structure.
user-invocable: false
---

# Estructura de `data/` en Next.js (fuente de datos JSON local)

Guía de referencia para proyectos JAMstack chicos (catálogos, vitrinas, sitios sin backend propio) que usan archivos JSON como fuente de verdad del contenido, en vez de un CMS o base de datos.

## Cuándo aplica este skill

`data/` con JSON local tiene sentido cuando:
- El catálogo es chico (decenas, no miles de ítems).
- El contenido lo actualiza quien programa el sitio, no un usuario de negocio sin conocimientos técnicos.
- No hay necesidad de un panel de administración.

Si en algún momento cualquiera de estas tres condiciones deja de cumplirse, es momento de migrar a Google Sheets (edición no técnica simple) o un CMS headless — ver sección de límites. **Nota: si el proyecto ya nació usando Google Sheets u otra fuente externa en vez de JSON local (confirmalo en el CLAUDE.md del proyecto), este skill sirve como referencia del contrato de datos (types/) pero la implementación de lectura real vive en nextjs-lib-structure, no acá.**

## Qué va dentro de `data/`

Archivos `.json` con el contenido real del sitio: productos, categorías, testimonios, etc. **Solo datos, nunca lógica.** `data/` no tiene funciones, no tiene JSX, no importa nada — son archivos planos que `lib/` lee.

## Estructura recomendada

```
data/
├── products.json
└── categories.json
```

## Mejores prácticas

### 1. La forma del JSON debe calzar 1:1 con el tipo en `types/`

```json
// data/products.json
[
  {
    "id": "1",
    "slug": "producto-ejemplo",
    "name": "Nombre del producto",
    "description": "Descripción del producto...",
    "price": 100,
    "images": ["https://res.cloudinary.com/.../imagen-1.jpg"],
    "category": "lenceria"
  }
]
```

```ts
// types/product.ts — debe calzar exactamente con el JSON de arriba
export interface Product {
  id: string
  slug: string
  name: string
  description: string
  price: number
  images: string[]
  category: string
}
```

### 2. `slug` único, en minúsculas, con guiones — pensado para URL

```json
"slug": "producto-ejemplo"
```

No: `"Producto Ejemplo (Talle M)"`.

### 3. `id` estable, `slug` puede cambiar

Usar un `id` que no cambie nunca, independiente del `slug`.

### 4. Rutas de imágenes consistentes

Si las imágenes son locales: siempre empezar con `/` (relativo a `public/`). Si son externas (ej. Cloudinary): URL completa y estable, nunca un link que pueda vencer o requerir autenticación.

### 5. Validar el JSON antes de commitear (JSON válido, sin comas colgantes)

Un JSON mal formado rompe el build completo.

### 6. Mantener el archivo ordenable y legible

Para un catálogo chico, mantener los productos en un orden consistente facilita encontrar y editar un producto específico a mano.

## Limitantes a tener en cuenta

- **No hay validación en runtime.** Si alguien edita el JSON a mano y comete un error de tipo, TypeScript no lo detecta en runtime.
- **Cada cambio requiere rebuild/redeploy** (si la fuente es JSON local leído en build time).
- **No apto para múltiples editores no técnicos.** Si alguien del negocio necesita agregar productos sin depender de un desarrollador, `data/` con JSON deja de ser suficiente — corresponde migrar a Google Sheets o un CMS.

## Cuándo migrar de `data/` (JSON) a otra fuente

| Señal | Migrar a |
|---|---|
| Alguien no técnico necesita editar el catálogo | Google Sheets (más simple) o CMS headless |
| El catálogo supera ~100-200 productos | CMS headless |
| Se necesita autenticación, roles, o flujo de aprobación de contenido | CMS headless |
| Se necesita que el sitio se actualice sin rebuild completo | Fuente con ISR (`revalidate`) — Sheets o CMS |

Importante: gracias a que `lib/` es el único lugar que sabe leer `data/` (o la fuente que sea), esta migración solo toca `lib/` — `app/`, `components/` y `types/` no deberían cambiar (ver `nextjs-lib-structure`).

## Don'ts (errores comunes)

- ❌ No poner lógica ni cálculos dentro del JSON — el JSON es dato crudo, cualquier cálculo va en `lib/`.
- ❌ No duplicar el mismo producto en dos archivos JSON distintos.
- ❌ No dejar `slug` duplicados entre productos.
- ❌ No mezclar catálogo con contenido no relacionado en el mismo archivo.
- ❌ No editar `data/` directo en producción sin pasar por control de versiones (Git).

## Skills relacionados

- `nextjs-lib-structure` — único lugar que debe leer `data/` o la fuente externa equivalente
- `nextjs-types-structure` — define la forma exacta que debe tener cada objeto
