# Usar un LLM como agente ofensivo — lecciones empíricas

## Cobertura determinística > agente decidiendo qué correr

Un catálogo grande de herramientas de ataque disponibles para que un agente LLM las invoque a discreción tiene cobertura NO confiable — se demuestra corriendo el mismo scope dos veces y viendo qué herramienta corrió una vez y la otra no. La ganancia de "poder" más alta por esfuerzo casi nunca es escribir detección nueva — es promover herramientas ya escritas y ya validadas contra falsos positivos a un sweep determinístico que SIEMPRE corre, quitándole la decisión al agente. El agente sigue siendo útil para lógica de negocio y encadenar hallazgos, no para decidir si una técnica básica se prueba o no.

## Modelos "sin censura" (uncensored) no son un atajo de recall

Probado empíricamente: un fine-tune "agresivo" del mismo tamaño que el modelo base, corrido contra el mismo target con el mismo scope, dio el mismo recall exacto que el baseline, pero:
- Tardó considerablemente más (en el orden de 3x).
- Generó un falso positivo NUEVO que el baseline no tuvo (confundió un fallo de tooling con un hallazgo real).
- Mostró loops de reintento sin avance que el baseline no mostró.

**Conclusión reusable:** "censura" del modelo base no era el cuello de botella de recall — el cuello de botella era cobertura determinística (ver punto anterior) y calidad de los confirmadores, no la disposición del modelo a "portarse mal". Antes de invertir en cambiar de modelo por recall, medir si el gap es de detección determinística faltante primero.

## El contrato de tool-calling es el filtro real para elegir modelo local

Al evaluar modelos especializados en seguridad para rol de agente (no de síntesis/Q&A), lo que determina si sirven no es el conocimiento del dominio sino si el runtime de inferencia local reporta soporte de tools correctamente Y el modelo realmente llena la estructura de tool-calls esperada en vez de emitir su propia sintaxis libre dentro del texto de respuesta. Un modelo puede:
- Reportar soporte de tools por heurística de arquitectura (falso positivo) y no llenar la estructura real — inútil como agente, entra en loops largos sin cerrar.
- Rechazar limpiamente por no soportar tools en absoluto (arquitectura base sin ese entrenamiento) — sigue siendo útil como modelo de completion/Q&A puro (explicar una vulnerabilidad conocida, sintetizar un resumen), nunca como agente que decide y ejecuta acciones.

**Regla práctica:** antes de adoptar un modelo especializado de seguridad como agente, correr un smoke-test con el schema de tools REAL del propio harness, no confiar en que el modelo "dice" soportar tools ni en benchmarks genéricos del modelo.

## RAG sobre fuentes de seguridad no depende de tool-calling

Un RAG (recuperación de conocimiento desde fuentes como bases de vulnerabilidades, catálogos de exploits, frameworks de técnicas, colecciones de payloads, guías de explotación) sigue siendo valioso como pieza independiente del agente — no necesita que el modelo subyacente soporte tool-calling, solo que pueda sintetizar bien un texto dado el contexto recuperado. Separar esta pieza del rol de "agente que decide y ejecuta" evita descartar todo el valor de una arquitectura solo porque una parte (el agente) no calificó.
