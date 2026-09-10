---
name: clean-code-practices
description: Principios de código limpio independientes del lenguaje o stack — SOLID, DRY, KISS, YAGNI, nombres, tamaño y responsabilidad de funciones, manejo de errores, estructura de código. Usar al escribir código nuevo (dev) o al revisarlo (qa), sin importar el lenguaje del proyecto.
user-invocable: false
---

# Principios de código limpio (independientes del stack)

Restatement original de principios ampliamente reconocidos en la industria (SOLID, Clean Code de Robert C. Martin, DRY/KISS/YAGNI) — no reproduce ningún texto ni ejemplo de ningún libro puntual, son las reglas en sí, que son de dominio del oficio.

## SOLID

- **SRP (responsabilidad única)** — una función/clase tiene una sola razón para cambiar. Si el nombre necesita "y" (`validateAndSave`), probablemente son dos.
- **OCP (abierto/cerrado)** — se puede extender comportamiento sin modificar el código ya probado y estable (ej. agregar un caso nuevo vía composición, no editando un `if/else` gigante ya existente).
- **LSP (sustitución de Liskov)** — si algo implementa una interfaz/contrato, debe poder usarse donde se espera ese contrato, sin sorpresas de comportamiento.
- **ISP (segregación de interfaces)** — interfaces chicas y específicas para quien las consume, no una gigante que obliga a implementar métodos que no se usan.
- **DIP (inversión de dependencias)** — depender de abstracciones (tipos, contratos), no de implementaciones concretas — es literalmente la regla de dirección única que ya aplica `nextjs-architecture-master` a nivel de capas.

## DRY / KISS / YAGNI

- **DRY** — misma lógica en dos lugares → extraer a una función compartida, no copiar/pegar con variaciones menores.
- **KISS** — la solución más simple que resuelve lo que pide la US, no la más "elegante" o genérica.
- **YAGNI** — no construir para un caso hipotético futuro que la US no pidió. Si hace falta después, se agrega después.

## Nombres

- Funciones: verbo + qué hacen (`getProductBySlug`, no `process` ni `doStuff`). Si el nombre necesita un comentario al lado para explicarse, el nombre está mal elegido.
- Variables: sustantivos descriptivos, sin abreviar salvo convención muy establecida (`i` en un loop está bien; `prd` por `product` no).
- Booleanos: forma de pregunta (`isAvailable`, `hasStock`, `canSubmit`), nunca un sustantivo pelado que se preste a confusión.
- Constantes: `SCREAMING_SNAKE_CASE` (`MAX_RETRY_COUNT`).

## Funciones

- Chicas: si no entra en una pantalla sin scrollear, sospechar — no es una regla dura de líneas, es una señal.
- Una función hace una cosa, en un solo nivel de abstracción (no mezclar "orquestar el flujo general" con "el detalle de un cálculo puntual" en la misma función).
- Pocos parámetros: máximo 3-4; más que eso, agrupar en un objeto/tipo.
- Sin efectos secundarios inesperados: una función que dice `getX` no debería, de paso, modificar estado global o escribir en disco.

## Estructura de código

- Guard clauses / early returns en vez de anidar `if` dentro de `if` dentro de `if` — más de 2-3 niveles de anidamiento es señal de refactor.
- Composición sobre configuración excesiva: preferir componer piezas chicas antes que una función/componente gigante con muchos parámetros booleanos controlando variantes.
- Cohesión: código relacionado, cerca (mismo archivo/carpeta); código no relacionado, separado — mismo criterio que ya aplican los skills de Next.js con la organización por dominio.

## Manejo de errores

- No silenciar errores (`catch` vacío). Como mínimo, loguear o propagar con contexto de qué operación falló.
- Fallar explícito y temprano — no dejar que un dato inválido se propague en silencio varias capas y explote lejos de su origen real.

## Comentarios

- El código explica el "qué". El comentario, cuando hace falta, explica el "por qué" de una decisión no obvia — no repite en palabras lo que la línea de abajo ya dice.

## Boy Scout Rule

Al tocar un archivo existente para esta US, si de paso ves algo pequeño y de bajo riesgo que viola estas reglas en las líneas que ya estás tocando, corregilo. No es licencia para refactors grandes fuera del alcance de la US — es dejar lo que tocaste un poco mejor de como lo encontraste, nada más.
