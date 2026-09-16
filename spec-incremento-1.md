# Spec — Incremento 1: Ingestión y extracción de estructura (un lenguaje)

## Referencias / Trazabilidad

Esta especificación no parte de cero. Cada afirmación de las secciones "Problema" y "Objetivo" está tomada de documentos ya existentes del proyecto, no inventada para este spec:

| Afirmación | Fuente | Estatus |
|---|---|---|
| Secuencia "un lenguaje → dos patrones → dataset pequeño validado → baseline → ..." | `SOLID_SEMILLERO_MASTER_CONTEXT.md`, sección de estrategia incremental | Autoritativo — citado casi literal |
| Necesidad de estructura confiable antes de cualquier `Finding` | `SOLID_SEMILLERO_MASTER_CONTEXT.md`, protocolo de detección | Autoritativo |
| Núcleo del producto (deuda técnica, énfasis en diseño/SOLID, enfoque híbrido) | Redefinición de producto acordada en esta conversación, basada en `ArticuloRevisado_SMS_LLM_SOLID.docx` y `ProyectopSOLID.pdf` | Autoritativo |

**Lo que este spec NO usa como fuente**, aunque existan en el proyecto:

- `ARQUITECTURA.pdf` y `DIAGRAMA_UML.pdf` — son propuestas del equipo, **no validadas**. Se mencionan en OPEN-004 únicamente como antecedente a considerar, no como restricción de este incremento. Tratarlas como fuente autoritativa aquí violaría la regla de evidencia de `constitution.md` (sección 7): ningún artefacto de diseño previo es válido por el solo hecho de existir.

## Problema

El sistema necesita poder leer el código fuente de un repositorio y obtener de él una representación estructural confiable (clases, métodos, relaciones) antes de que cualquier análisis de deuda técnica o principios SOLID sea posible. Sin esto, no hay evidencia sobre la cual construir ningún `Finding`.

## Objetivo del incremento

Demostrar que el pipeline de ingestión funciona de extremo a extremo para **un solo lenguaje**: dado un repositorio, extraer su estructura de código de forma determinística y reproducible, sin todavía intentar detectar ninguna violación de diseño.

Este es el primer eslabón de la estrategia incremental ya definida para el proyecto (un lenguaje → dos patrones → dataset pequeño validado → baseline → ...). No se adelanta ningún paso posterior de esa secuencia.

## Alcance

- Clonar o leer un repositorio local de un solo lenguaje de programación.
- Parsear los archivos fuente y construir un AST o representación equivalente.
- Extraer: clases, métodos, atributos, relaciones de herencia/composición básicas.
- Persistir esa estructura de forma que una fase posterior (detección) pueda consultarla.

## Fuera de alcance (en este incremento)

- Detección de cualquier violación SOLID o code smell.
- Cualquier llamada a un LLM.
- Soporte multi-lenguaje.
- Interfaz de usuario.
- Dataset de validación con gold standard (eso es el incremento siguiente de la secuencia).

## Requisitos

### REQ-001 Parsing de archivos fuente
**Contexto:** el sistema recibe la ruta de un repositorio local de **Python**.
**Evento/condición:** se ejecuta el proceso de ingestión sobre esa ruta.
**Comportamiento esperado:** el sistema identifica todos los archivos `.py` y los parsea sin error para el subconjunto de sintaxis Python válida (según la gramática Python que soporte Tree-sitter).
**Criterio de aceptación:** dado un repositorio de prueba conocido, el número de clases y métodos extraídos coincide con un conteo manual de referencia.

### REQ-002 Extracción de estructura
**Contexto:** un archivo `.py` fue parseado correctamente.
**Comportamiento esperado:** el sistema extrae clase(s), método(s) por clase, y relaciones de herencia declaradas explícitamente — **incluyendo herencia múltiple**, ya que Python la permite de forma nativa (`class C(A, B):`), a diferencia de lenguajes de herencia simple como Java.
**Criterio de aceptación:** la estructura extraída es serializable (JSON) y reproducible — correr el proceso dos veces sobre el mismo repo produce el mismo resultado.

### REQ-003 Manejo de errores de parsing
**Contexto:** un archivo no puede parsearse (sintaxis inválida, encoding no soportado, etc.).
**Comportamiento esperado:** el sistema registra el archivo como no procesado, con la razón, y continúa con el resto del repositorio — un archivo roto no detiene la ingestión completa.
**Criterio de aceptación:** un repositorio de prueba con al menos un archivo corrupto termina el proceso y reporta ese archivo como fallido, sin abortar los demás.

## Casos límite

- Repositorio vacío.
- Archivo de 0 bytes.
- Archivo con codificación distinta a UTF-8.
- Clases anidadas o clases definidas dentro de funciones (válido en Python).
- Herencia múltiple con orden de resolución (MRO) — no se resuelve el MRO en este incremento, solo se registran las clases base declaradas.
- Clases que heredan de `object` implícitamente (no declarado en el código) — no se cuenta como relación explícita.

## Preguntas abiertas

Ninguna pendiente. Las cuatro preguntas originales quedaron resueltas como DEC-001 a DEC-004 (ver sección Decisiones).

## Decisiones

- **DEC-001** — Persistencia de la estructura extraída: JSON simple para este incremento, no el modelo UML del equipo (sin validar todavía vía Plan). *Resuelve OPEN-004.*
- **DEC-002** — Tipo de repositorio de prueba: repo pequeño real de código abierto, **en Python**. La selección del repositorio específico se hace en Tasks — no bloquea el cierre de Specify.
- **DEC-003** — Lenguaje del código analizado ("paciente"): **Python**. Coherente con el stack de backend sugerido en `ARQUITECTURA.pdf` (LangGraph) y con la formación técnica del equipo (master context, sección 86). *Resuelve OPEN-001.*
- **DEC-004** — `ABC`/`Protocol` (equivalentes Python de "interfaz") se **excluyen** de la relación de herencia contada en este incremento; se tratan igual que herencia de clase normal, sin lógica especial. Se revisa en el incremento 2 si hace falta distinguirlas. *Resuelve OPEN-003.*

## Definition of done

REQ-001 a REQ-003 tienen evidencia verificable (criterios de aceptación cumplidos) y no quedan OPEN de severidad alta sin resolver. **Cumplida a nivel de especificación: las cuatro preguntas abiertas están resueltas (DEC-001 a DEC-004). Pendiente únicamente la evidencia de ejecución real (Test Gate), que llega en Implement.**
