# PRD — Advisor AI MultiAgente

## Motivación

El asesoramiento financiero de calidad sigue siendo caro, fragmentado y con barreras de acceso para gran parte del retail. Aprovechar LLMs dentro de una Agentic Composable Architecture permite diseñar una solución escalable, verificable y adaptable:

- **Agentes especializados:** cada agente gestiona una responsabilidad clara (ingesta, extracción de riesgo, filtrado de costes, generación de shortlist, explicación al cliente), facilitando validación reglamentaria, pruebas A/B y auditoría por componente.
- **Arquitectura modular composable:** los módulos pueden recombinarse o sustituirse sin rehacer el sistema, acelerando iteraciones, despliegues por fases y adaptación ante cambios regulatorios o nuevos modelos LLM.
- **Robustez operativa:** separación de responsabilidades que limita el blast radius ante fallos y facilita controles de cumplimiento centralizados, versionado de fuentes y trazabilidad de decisiones.
- **Explicabilidad y grounding:** agentes de grounding garantizan que todas las afirmaciones estén respaldadas por citas verificables (KID/DFI/folleto), reduciendo riesgo regulatorio y mejorando transparencia.
- **Escalabilidad de producto:** capacidades avanzadas (optimización cuantitativa, reporting regulatorio, alertas ESG) se integran como agentes adicionales sin reescribir el núcleo.

**Beneficio práctico:** producir propuestas de inversión rápidas, trazables y adaptativas, reducir costes operativos e impulsar validación temprana de producto y cumplimiento.

---

## Objetivo

Entregar un MVP operativo que en entorno educativo genere en menos de **10 segundos** para cada usuario **3 carteras modelo adaptativas** justificadas y trazables, con al menos **3 citas** por propuesta extraídas exclusivamente de KID/DFI y folletos públicos, y que produzca un PDF exportable con envío por email.

**Flujo base:** ingestion → RAG (vector store) → LLM  
**Residencia de datos:** UE  
**Registro:** para auditoría

### Criterios medibles

- Latencia de propuesta < 10 s  
- PDF final < 2 MB  
- ≥ 3 citas por cartera  
- Recuperación por ISIN/gestora ≥ 5 fragments relevantes  
- Consentimiento registrado; PII minimizada; datos en UE

---

## Descripción funcional de la herramienta

### Ingestor de documentos

> Ver sección detallada más abajo

### Preprocesador y extractor

- **Propósito:** limpieza, OCR fallback, extracción de texto y segmentación en fragments con metadatos  
- **Salidas:** chunks con metadatos `{docId, tipo, url, fecha_doc, página, posible_ISIN}`

### Indexador RAG

- **Propósito:** generar embeddings por fragmento y permitir recuperación semántica filtrable por metadatos  
- **Salidas:** índice consultable por agentes

### Onboarding y perfilado conversacional

- **Propósito:** capturar liquidez, tolerancia a pérdida, horizonte, objetivo y experiencia; mapear a perfil y SRRI/SRI objetivo  
- **Salidas:** perfil estructurado y consentimiento registrado

### Recuperador de contexto y shortlist

- **Propósito:** según perfil, recuperar fragmentos relevantes (riesgo, costes, política, liquidez) y construir shortlist de fondos candidatos con evidencia  
- **Salidas:** lista priorizada de fondos con citas

### Generador de carteras modelo adaptativas

- **Propósito:** aplicar plantillas core‑satellite y reglas por perfil para producir 3 alternativas: coste, calidad y ajuste macro  
- **Salidas:** 3 carteras con pesos, verificación de límites y lista de citas

### Motor de explicación y trazabilidad

- **Propósito:** redactar en lenguaje claro el “por qué” de cada cartera, anclando afirmaciones a citas verificables  
- **Salidas:** texto explicativo con ≥ 3 citas por cartera

### Generación de PDF y entrega por email

- **Propósito:** componer informe con portada, perfil, carteras, explicaciones, citas y disclaimers; envío y registro  
- **Salidas:** PDF y registro de envío

### Observabilidad y cumplimiento

- **Propósito:** logs de ingestion, indexación, consultas RAG, respuestas LLM y envíos; métricas de calidad y panel básico de supervisión  
- **Salidas:** métricas y logs para auditoría

### UX administrativa y operaciones

- **Propósito:** controlar fuentes, forzar reindexado, revisar documentos y aprobar disclaimers; operativa transversal

### Fases posteriores

- **Fase 2:** Optimización cuantitativa; base de datos estructurada con métricas históricas; integración EMT target market  
- **Fase 3:** Reporting regulatorio; versionado formal; alertas ESG; multilingüe

---

## Ingestor de documentos — Documentos mínimos desde la CNMV

**Objetivo:** definir el conjunto mínimo de documentos públicos de fondos que el Ingestor debe localizar y descargar desde la CNMV para que el MVP pueda recuperar evidencia suficiente y cumplir los requisitos de trazabilidad y explicabilidad.

### Documentos mínimos (ordenados por prioridad)

#### 1. KID / DFI — Mínimo obligatorio

- **Por qué:** contiene SRRI/SRI, perfil de riesgo, resumen de costes, política de inversión resumida, liquidez  
- **Datos clave:** SRRI/SRI, comisiones (TER, suscripción/reembolso), política de inversión, liquidez, fecha_doc, página

#### 2. Folleto / Prospectus — Mínimo obligatorio

- **Por qué:** política de inversión completa, límites, derivados, comisiones, cláusulas legales  
- **Datos clave:** política detallada, uso de derivados, restricciones, estructura de costes, gestora, fecha_doc, páginas

#### 3. Ficha técnica / Fact Sheet — Mínimo recomendado

- **Por qué:** AUM, rentabilidades, benchmark, antigüedad  
- **Datos clave:** AUM, fecha de lanzamiento, rentabilidades (1y/3y/5y), benchmark, ISIN, fecha_doc

### Documentos opcionales para iteraciones futuras

- Informes anuales / semestrales  
- Prospectus supplements  
- Desglose TER  
- Documentos ESG / sostenibilidad

### Razonamiento funcional mínimo

- El MVP exige evidencia en tres dominios: **Riesgo**, **Costes**, **Política de inversión**  
- KID/DFI y Folleto cubren los tres; la Ficha mejora calidad de selección  
- Si sólo hay dos documentos, el sistema debe justificar con lo disponible

### Metadatos mínimos por documento

- `docId`, `tipo`, `url_origen`, `fecha_doc`, `fecha_descarga`, `checksum`, `paginas`, `ISINs_detectados`, `gestora`, `calidad_texto`

### Metadatos por fragmento (chunk)

- `chunkId`, `docId`, `tipo`, `url`, `fecha_doc`, `página`, `offset_texto`, `posible_ISIN`, `etiqueta_tematica`, `score_calidad_OCR`

### Política de ingestión mínima

- Prioridad: KID/DFI > Folleto > Ficha  
- Versionado obligatorio  
- Si no hay ISIN, marcar como “sin identificación”  
- Revisión manual si necesario

### Guía de alcance para próximas iteraciones

- Iteración 2: añadir informes anuales y supplements  
- Iteración 3: integrar fuentes externas (gestoras, EMT, datos abiertos); normalizar campos numéricos

---

## Plan del MVP alto nivel

- **Semana 1–2:** Ingesta + OCR + RAG; cuestionario y derivación de perfil  
- **Semana 3–4:** Reglas de selección y plantillas por perfil; generación de 3 carteras; explicabilidad con citas  
- **Semana 5:** PDF + email; panel mínimo de observabilidad; pruebas E2E; cierre MVP

---

## Parte técnica

### Arquitectura técnica

- **Patrón general:** arquitectura modular basada en agentes orquestados por un coordinador ligero  
- **Capas principales:** ingestión, procesamiento, RAG, agentes LLM, orquestación, almacenamiento, frontend, entrega  
- **Resiliencia:** versionado por agente, logs estructurados, fallbacks y modos degradados

### LLM y red de agentes

- El LLM actúa como motor de razonamiento, organizado en agentes especializados:
  - Ingestor
  - Preprocesador
  - Indexador
  - Perfilado
  - Recuperador
  - Selector
  - Generador de Carteras
  - Explicador
  - PDF/Entrega
  - Observabilidad
  - Admin Ops

- Cada agente versiona sus prompts y outputs; registra fragments citados y metadatos de confianza

### Integraciones y componentes

- **Fuentes:** CNMV (KID/DFI, folletos) y webs de gestoras  
- **Vector store:** colección por `tipo=KID|Folleto|Ficha`; metadatos `{docId, url, fecha_doc, página}`  
- **PDF:** plantilla con portada, resumen, carteras, citas y disclaimers  
- **Email:** SMTP/Graph básico para confirmación y logs

### Seguridad y datos

- **PII mínima:** nombre y email opcionales; consentimiento explícito; cif
