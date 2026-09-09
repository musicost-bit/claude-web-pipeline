---
name: user-story-format
description: Plantilla y checklist para escribir una User Story completa, y cuándo agruparlas bajo un EPIC en vez de escribir una US gigante. Usar siempre que haya que redactar una User Story o decidir si un pedido debe trocearse en varias.
user-invocable: false
---

# Formato de User Story

## Cuándo usar EPIC vs US directa
Un pedido específico y acotado (una pantalla, un botón, un ajuste puntual) va directo a US.
Un pedido amplio o que menciona un sitio de referencia ("quiero un sitio como X") casi siempre esconde varias piezas independientes — trocealo en un EPIC + varias US hijas. Señal de alerta: si no podés escribir un solo set de criterios de aceptación sin que la lista supere 5-6 escenarios, es más de una US.

## Plantilla EPIC
Archivo: `docs/user-stories/EPIC-<NN>-<slug>.md`

    # EPIC-<NN>: <resumen corto del pedido original>

    ## Pedido original
    [Tal como lo dijo el cliente, textual]

    ## Referencia
    [URL si la dio, y qué se rescató tras inspeccionarla]

    ## Conversación
    [Resumen del ida y vuelta: qué preguntó el agente, qué contestó el cliente]

    ## Alertas de alcance
    [Lo que chocó con las reglas del proyecto y cómo se resolvió]

    ## User Stories derivadas
    - [ ] US-<NN>-<slug> — <título corto>

## Plantilla US
Archivo: `docs/user-stories/US-<NN>-<slug>.md`

    ## Título
    [Verbo + objetivo, una línea]

    ## Contexto
    [2-4 líneas: qué pidió el usuario, por qué]

    ## Historia
    Como [rol]
    Quiero [funcionalidad]
    Para [beneficio]

    ## Áreas técnicas afectadas
    [Qué capas/módulos toca, según la arquitectura documentada de este proyecto]

    ## Criterios de aceptación (Gherkin)
    - Given [contexto] When [acción] Then [resultado]
    (mínimo 2-3, incluyendo un caso borde: sin resultados, dato faltante, error)

    ## Consideraciones UX
    [Estados de carga/vacío/error, responsive, accesibilidad — vacío si no aplica]

    ## Fuera de alcance
    [Explícito]

    ## Preguntas resueltas
    [Qué preguntó el agente y qué contestó el cliente]

    ## Definition of Done
    - [ ] Código implementado y con tests
    - [ ] Pasa build/lint/tipos
    - [ ] Cumple la arquitectura documentada del proyecto
    - [ ] PR creado y revisado

## Numeración y carpeta
Todo va en `docs/user-stories/`. Numerá con dos dígitos (`01`, `02`...) en orden de creación — no reutilices ni saltees números, aunque una US se rechace.
