# Architecture Drivers — Incremento 1 (Ingestión y extracción de estructura)

## 1. Contexto arquitectónico

Este documento no propone solución — es la destilación de qué condiciona la arquitectura del Incremento 1, a partir de `spec-incremento-1.md`, `constitution.md` y `ADR-001-backend-django.md`. La solución candidata se decide en la siguiente sub-fase (Architecture Synthesis).

## 2. Drivers funcionales

- **FD-001** — Parsear archivos `.py` y extraer clases/métodos (REQ-001, REQ-002).
- **FD-002** — Manejar errores de parsing sin abortar el proceso completo (REQ-003).

## 3. Atributos de calidad relevantes

| Atributo | Prioridad | Origen |
|---|---|---|
| Reproducibilidad | Alta | REQ-002: correr el proceso dos veces produce el mismo resultado |
| Modificabilidad / extensibilidad multi-lenguaje | Alta | Visión de producto confirmada: analizador multilenguaje, aunque este incremento solo cubre Python |
| Robustez ante entradas inválidas | Alta | REQ-003 y casos límite (archivo corrupto, encoding no UTF-8, repo vacío) |

## 4. Restricciones aplicables

- El backend es Django/Python (`ADR-001`) → cualquier parser elegido debe ser invocable desde Python.
- `constitution.md`, sección 2: los resultados determinísticos y las inferencias del LLM se mantienen diferenciados en todo momento. Este incremento es 100% determinístico (parsing) — coherente con "Fuera de alcance: cualquier llamada a un LLM" del spec.

## 5. ASRs derivados

### ASR-001 — Extensibilidad multi-lenguaje
**Origen:** FD-001 + visión de producto (analizador multilenguaje).
**Impacto arquitectónico:** el mecanismo de parsing debe poder extenderse a otros lenguajes sin rediseñar el pipeline de ingestión completo. Favorece un parser generador/multi-gramática sobre uno atado a un solo lenguaje.

### ASR-002 — Reproducibilidad
**Origen:** REQ-002, criterio de aceptación.
**Impacto arquitectónico:** el proceso no puede depender de estado no determinístico (p. ej. orden de iteración del sistema de archivos, que no está garantizado) — hay que ordenar explícitamente los archivos antes de procesar.

### ASR-003 — Aislamiento de lo determinístico frente a lo probabilístico
**Origen:** `constitution.md` sección 2 + "Fuera de alcance" del spec.
**Impacto arquitectónico:** el módulo de parsing debe quedar desacoplado de cualquier módulo LLM — sin dependencia de import ni acoplamiento de datos que asuma la presencia del LLM.

## 6. Alternativas de parser a evaluar en Architecture Synthesis

*(no se decide aquí — se lista para la siguiente sub-fase)*

- **Tree-sitter** (con bindings Python) — ya mencionado en `ARQUITECTURA.pdf` del equipo, pero **no validado** contra estos ASRs todavía.
- **Módulo `ast` nativo de Python** — sin dependencias externas, pero solo parsea Python (no satisface ASR-001 por sí solo).
- **`libcst`** — preserva formato y comentarios; más pesado que `ast`.

## 7. Riesgos y supuestos

- Tree-sitter aparece en la arquitectura previa del equipo, pero eso no lo convierte en la elección válida — se evalúa igual que las demás alternativas en Architecture Evaluation (`constitution.md`, sección 7: ningún artefacto de diseño previo es válido por el solo hecho de existir).
