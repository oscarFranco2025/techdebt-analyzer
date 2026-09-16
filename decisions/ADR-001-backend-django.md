# ADR-001 — Framework de backend: Django

## Estado
Aceptado

## Contexto

El backend del tool ("el médico") necesita un framework de implementación. La arquitectura del equipo (`ARQUITECTURA.pdf`, Capa 2) ya sugería Python de forma indirecta al usar LangGraph para orquestación, y DEC-003 (`spec-incremento-1.md`) fijó Python como lenguaje del código que se analiza ("el paciente") — son decisiones independientes entre sí, pero mantener ambos lados en Python reduce la complejidad operativa de un equipo pequeño.

## Alternativas consideradas

Tomadas de las tecnologías candidatas ya listadas en `SOLID_SEMILLERO_MASTER_CONTEXT.md` (sección 26.2):

| Alternativa | Ventajas | Costos / riesgos |
|---|---|---|
| **Django** (elegida) | Framework "baterías incluidas" — ORM, admin panel, autenticación listos; el equipo ya lo domina | Soporte async del ORM históricamente más lento que frameworks async-nativos; trae piezas (templates, auth de sesión) que este proyecto puede no necesitar completas |
| FastAPI | Async-nativo desde el diseño; más común en servicios de IA/ML; ligero | El equipo no tiene la misma experiencia previa; sin ORM ni admin panel incluidos — hay que construirlos o agregar SQLAlchemy aparte |
| Node.js / NestJS | Mismo lenguaje (TypeScript) que el frontend Angular, stack unificado | Rompe la coherencia con LangGraph/Python ya sugerida en la arquitectura; el equipo tiene menos experiencia aquí que en Django |
| .NET | Tipado fuerte, buen tooling empresarial | Menor alineación con el ecosistema Python de IA/ML (LangGraph, Transformers, etc.); sin evidencia de experiencia previa del equipo |

## Decisión

Se elige **Django**.

## Justificación

- **Experiencia previa del equipo** — razón principal y suficiente dado el principio de proporcionalidad: para un semillero de investigación con tiempo limitado, reducir la curva de aprendizaje reduce directamente el riesgo de ejecución.
- El ORM y el admin panel de Django ofrecen CRUD gratuito sobre el modelo de datos que el equipo ya había propuesto (`Project`, `Analysis`, `Finding`, `Evidence`, etc.) sin construir una interfaz de administración a mano — útil en las fases tempranas de investigación para inspeccionar datasets y experimentos.

## Consecuencias / riesgos a monitorear (no bloqueantes)

- El soporte async nativo del ORM de Django ha madurado (métodos como `aget`/`acreate`/`afilter`), pero ha ido históricamente por detrás de frameworks async-nativos. Como la orquestación con LangGraph y las llamadas al LLM son naturalmente asíncronas e IO-bound, esto merece atención explícita cuando Plan diseñe la integración `AnalysisService` ↔ Django — no es un impedimento conocido hoy, es un punto a vigilar.
- Queda pendiente para Plan decidir si se usa Django REST Framework para exponer la API REST ya dibujada en `ARQUITECTURA.pdf`, dado que la Presentación (Angular) es un cliente separado, no vistas server-side de Django.

## Relacionado

- `ARQUITECTURA.pdf`, Capa 2 (orquestación `AnalysisService` / LangGraph)
- `spec-incremento-1.md`, DEC-003 (lenguaje del código analizado: Python)
- `SOLID_SEMILLERO_MASTER_CONTEXT.md`, sección 26.2 (tecnologías candidatas de backend)
