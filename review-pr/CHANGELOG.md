# Registro de Versiones (Changelog) - review-pr

## v1
**Funcionalidad principal actual:**
- **Revisión Silenciosa:** Opera sin generar ruido en chat (Agente Silencioso), entregando solo el reporte directivo al final.
- **Enrutamiento Dinámico:** Interactúa automáticamente con la skill `stack-detector` para comprender la arquitectura antes de revisar el código.
- **Reglas Estrictas Anti-Alucinación:** Tiene rotundamente prohibido emitir recomendaciones genéricas o de industria común; obliga al agente a leer el `usage.md` de la skill del framework específico e iterar sobre esas reglas exactas.
- **Memoria con Engram (Fases 2.0 y 6.5):** Busca el ID del PR antes de iniciar para no duplicar feedback y, al terminar, guarda un sumario local (memoria) persistente en el equipo.
- **Cálculo Financiero (Fase 7):** Realiza aproximaciones matemáticas (contando palabras in/out) para dar un estimado realista de gasto en USD del LLM.
- **Filtro Seguro de Diff:** Excluye dinámicamente binarios, tests, JSON y SVG localmente, leyendo únicamente código añadido (`+`) para el `inline feedback` usando Bitbucket API.