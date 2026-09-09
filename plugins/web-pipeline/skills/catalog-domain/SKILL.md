---
name: catalog-domain
description: Reglas de negocio para sitios de catálogo/vitrina de productos con contacto directo (WhatsApp u otro canal) en vez de compra en línea. Usar siempre que una User Story toque catálogo, ficha de producto, o cualquier acción del visitante sobre un producto.
user-invocable: false
---

# Dominio: catálogo de productos con contacto directo

## Patrón de negocio
Sitio vitrina para mostrar productos y derivar el contacto a WhatsApp (u otro canal directo) — no es un ecommerce transaccional. El patrón es el mismo sea cual sea el rubro (tapizados, lencería, muebles...): cambia el producto, no el modelo de negocio.

El rubro específico, la fuente de datos elegida, y el modelo exacto de campos de ESTE proyecto están en su CLAUDE.md — leelo antes de aplicar estas reglas a un pedido concreto.

## Reglas que nunca se rompen
Si un pedido menciona cualquiera de estos, es un conflicto de alcance — señalalo en la conversación antes de convertirlo en criterio de aceptación:
- Carrito de compras
- Checkout / pasarela de pago / tarjeta de crédito o débito
- Cuenta de usuario / login / registro
- Sincronización de inventario en tiempo real (si el pedido real es "que alguien no-técnico edite el catálogo fácil, sin tocar código", ESO no es esto — ver la tabla de fuentes de datos más abajo, ya cubre Excel/Drive)
- CMS o panel de administración con backend propio

## Qué sí es parte normal del alcance
- Navegar/filtrar productos por categoría o los campos que tenga la fuente de datos
- Ver el detalle de un producto (galería, descripción, atributos propios del rubro)
- Botón de contacto directo con mensaje prellenado mencionando el producto
- Páginas informativas: sobre nosotros, ubicación, horario
- Favoritos/comparar guardado en el navegador (localStorage) — sin cuenta ni backend

## Fuente de datos — varía por proyecto
| Fuente | Cuándo se usa | Implicancia para la US |
|---|---|---|
| JSON local | Catálogo chico, nadie no-técnico lo edita | Editar = tocar el archivo y regenerar (SSG) |
| Excel/CSV | El dueño del negocio quiere editar sin tocar código | Necesita conversión a JSON en build time, o script de sync |
| Google Drive (planilla + carpeta de imágenes) | El cliente ya administra ahí su catálogo | Necesita credenciales de API (Sheets/Drive) — evaluar si el build sigue siendo 100% estático |

Las imágenes siguen a la fuente de datos: JSON local → `public/` con `next/image`; Excel/Drive → definir si se copian a `public/` en build time o se sirven desde URL externa (Drive, Cloudinary).

## Modelo de producto
No se enumera acá — es responsabilidad de `types/<entidad>.ts` del proyecto (ver nextjs-data-structure / nextjs-types-structure si el stack es Next.js). Un producto siempre tiene identificador, nombre, categoría, precio, imágenes y descripción; cualquier campo adicional (material, color, medidas, disponibilidad) está en el tipo real del proyecto, no acá — evita que este skill y el tipo se desincronicen.
