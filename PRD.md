
# PRD — Advisor AI MultiAgente

**Versión:** 1.0 
**Clasificación:** Pública  
**Fecha:** 2025-09-26  
**Propósito:** Definir el **MVP** (flujo funcional simplificado con RAG sin BBDD estructurada), requisitos funcionales/técnicos, roadmap y primeras tareas detalladas de la Fase 1.

> **Aviso**: Este sistema es **educativo** y **no constituye asesoramiento financiero**. Se basa en documentación pública (KID/DFI, folletos, fichas) y buenas prácticas de idoneidad MiFID para lenguaje claro y registro, sin sustituir obligaciones reguladas. Véanse referencias: CNMV sobre DFI/KID, ESMA (MiFID idoneidad), metodología SRRI/SRI (UCITS/PRIIPs), EMT FinDatEx y GDPR.  
> Referencias: [CNMV–DFI/KID](https://www.cnmv.es/Portal/inversor/Fondos-DFI?lang=es), [ESMA MiFID II Suitability 2022](https://www.esma.europa.eu/press-news/esma-news/esma-publishes-final-guidelines-mifid-ii-suitability-requirements-0), [CESR/ESMA SRRI 10-673](https://www.esma.europa.eu/sites/default/files/library/2015/11/10_673.pdf), [ESAs PRIIPs Q&A consolidado](https://www.esma.europa.eu/sites/default/files/2023-05/JC_2023_22_-_Consolidated_JC_PRIIPs_Q_As.pdf), [FinDatEx EMT](https://www.findatex.eu/), [GDPR – EUR‑Lex](https://eur-lex.europa.eu/eli/reg/2016/679/oj/eng).

---

## 1) Objetivos
- Generar **3 carteras modelo** para cliente retail a partir de su **perfil** y **documentación pública** de fondos registrados en la **CNMV**.  
- Mantener **trazabilidad** mediante **citas textuales** a KID/DFI y folletos (modo *grounded‑only*).  
- Entregar **explicación clara**, **PDF** y **envío por email**.  
- **MVP sin BBDD estructurada**: ingestion → RAG (vector DB) → LLM.

## 2) Alcance (MVP)
- **Usuarios**: cliente retail (entorno educativo).  
- **Productos**: fondos UCITS registrados en CNMV (España/UE).  
- **Canal**: web (chat conversacional en español, EUR).  
- **Deliverables**: 3 carteras + explicación + PDF + email.  
- **Quedan fuera** (iteraciones posteriores): órdenes de suscripción, firma, CRM, optimización cuantitativa avanzada, reporting regulatorio completo.

---

# PARTE FUNCIONAL

## 3) Flujo funcional del **MVP** (sin BBDD estructurada)
1. **Descubrimiento y descarga automática** de PDFs (KID/DFI, folletos, fichas) desde fuentes públicas (CNMV/gestoras). *(Mensual y bajo demanda).*  
2. **Preprocesamiento**: limpieza, OCR si es necesario, y segmentación en **fragmentos** (*chunks*) con metadatos mínimos `{docId, tipo, url, fecha_doc, página, posible ISIN}`.  
3. **Indexación RAG (Vector DB)**: se calculan embeddings de cada fragmento y se indexan para recuperación semántica.  
4. **Onboarding** (chat) y **perfilado simple**: liquidez, capacidad de pérdida, horizonte, objetivo, experiencia → perfil (Conservador/Moderado/Dinámico/Agresivo) y **SRRI/SRI objetivo**. *(MiFID idoneidad en lenguaje claro)*.  
5. **Recuperación de contexto** (RAG): dado el perfil, el agente busca fragmentos relevantes (riesgo 1–7, costes, política, liquidez) por categoría para construir un *shortlist* de fondos candidatos (no se persisten métricas estructuradas).  
6. **Propuesta de carteras** (regla simple): plantilla **core‑satellite** por perfil + límites básicos (coste/antigüedad/AUM aproximados según citas) → **3 alternativas**: (A) coste, (B) calidad/consistencia, (C) ajuste ligero a contexto macro.  
7. **Explicación y citas**: el LLM redacta razones en lenguaje claro y añade **citas** `[tipo, página, fecha_doc]` a pasajes del KID/folleto/ficha.  
8. **Generación de PDF y envío por email**; registro simple de la interacción y lista de documentos citados.

## 4) Requisitos funcionales
- **Onboarding**: cuestionario llano; derivación de perfil y SRRI/SRI objetivo (1–7).  
- **Criterios de selección básicos** (a nivel documental, sin cálculo): excluir clases con comisiones de entrada, **TER elevado** (comparado vs. pasajes citados), antigüedad muy baja, o complejidad no apta retail.  
- **Límites de diversificación** por cartera: máx. 25% por fondo, máx. 40% por gestora.  
- **Explicabilidad**: al menos **3 citas** por propuesta (riesgo, costes, política).  
- **Disclaimers** visibles en UI y PDF.  
- **No bloqueo** por faltantes: si falta un dato crítico, se **sugiere alternativa** o se marca la asunción de forma transparente.

## 5) Requisitos no funcionales
- **Privacidad** y residencia de datos en **UE** (GDPR); minimización de PII; consentimiento.  
- **Disponibilidad** objetivo 99,5% (MVP); **rendimiento**: primera propuesta < 10 s; PDF < 5 s.  
- **Observabilidad mínima**: logs de descargas, indexación y citas; métrica % propuestas con ≥3 citas.

## 6) Cumplimiento (enfoque educativo)
- Uso de **KID/DFI** y **Folleto** como fuentes primarias, lenguaje claro, y registro de respuestas (buenas prácticas **MiFID II**).  
- **Indicador 1–7** SRRI/SRI según documentos públicos (no recalculado).  
- **grounded‑only**: no se afirman datos sin cita.  
- Referencias: [CNMV–DFI/KID](https://www.cnmv.es/Portal/inversor/Fondos-DFI?lang=es), [ESMA MiFID II Suitability](https://www.esma.europa.eu/press-news/esma-news/esma-publishes-final-guidelines-mifid-ii-suitability-requirements-0), [SRRI CESR/10‑673](https://www.esma.europa.eu/sites/default/files/library/2015/11/10_673.pdf), [PRIIPs Q&A](https://www.esma.europa.eu/sites/default/files/2023-05/JC_2023_22_-_Consolidated_JC_PRIIPs_Q_As.pdf).

---

# PARTE TÉCNICA (MVP)

## 7) Arquitectura técnica
- **Ingestor ligero** (descarga + OCR opcional).  
- **Procesador de documentos** (segmentación a *chunks* con metadatos mínimos).  
- **Vector DB** (RAG) para recuperación semántica.  
- **Servicio LLM** con herramientas: búsqueda RAG, generador de carteras (regla simple), generador de PDF.  
- **Frontend web** (chat + vista de carteras y “por qué”).

## 8) Integraciones y componentes
- **Fuentes**: CNMV (KID/DFI, folletos) y webs de gestoras (fichas).  
- **Vector store**: colección por `tipo=KID|Folleto|Ficha`; metadatos `{docId, url, fecha_doc, página}`.  
- **Generación de PDF**: plantilla con portada, resumen, carteras, citas y disclaimers.  
- **Email**: servicio SMTP/Graph básico (confirmación y log).

## 9) Seguridad y datos
- **PII mínima** (nombre y email opcionales); consentimiento explícito; cifrado en tránsito y reposo; almacenamiento UE (GDPR).  
- **Logging**: guardar prompts/respuestas con **metadatos y citas**, nunca *chain‑of‑thought*.

---

# 10) Roadmap
- **Fase 1 (MVP)** — RAG + LLM sin BBDD estructurada, reglas simples de cartera, PDF/email.  
- **Fase 2** — **Optimización cuantitativa** (p. ej., mínima varianza, volatilidad objetivo o *tracking error* a benchmark) y **target market EMT** cuando esté disponible públicamente.  
- **Fase 3** — **Reporting regulatorio** (costes ex‑ante, anexos, versionado formal), **BBDD estructurada** de métricas históricas, ESG, alertas por evento, multilingüe.  

Referencias de marco: [FinDatEx EMT](https://www.findatex.eu/), [ESMA/PRIIPs](https://www.esma.europa.eu/sites/default/files/2023-05/JC_2023_22_-_Consolidated_JC_PRIIPs_Q_As.pdf).

---

# 11) Plan del **MVP** (alto nivel)
- **Semana 1–2**: Ingesta + OCR + RAG; cuestionario y derivación de perfil.  
- **Semana 3–4**: Reglas de selección y plantillas por perfil; generación de 3 carteras; explicabilidad con citas.  
- **Semana 5**: PDF + email; panel mínimo de observabilidad; pruebas E2E; cierre MVP.

---

# 12) Desarrollo MVP — **Primeras tareas (Fase 1)**

## Tarea 1 — Ingesta & RAG mínimos
**Objetivo**: disponer de documentos indexados para citas.  
**Funcional**: descargar KID/DFI/folletos/fichas, OCR si procede, *chunking*, embeddings, indexación.  
**Técnico**: job manual y por cron mensual; metadatos `{docId, url, fecha_doc, página}`; política de prioridad: KID/DFI > Folleto > Ficha.  
**Aceptación**: consultar por ISIN/gestora y recuperar ≥5 fragmentos relevantes con página y fecha_doc.

## Tarea 2 — Onboarding y perfilado
**Objetivo**: obtener perfil (Conservador/Moderado/Dinámico/Agresivo) y SRRI/SRI objetivo.  
**Funcional**: cuestionario llano (liquidez, capacidad de pérdida, horizonte, objetivo, experiencia).  
**Técnico**: formulario en chat; reglas de puntuación y mapeo a rango SRRI/SRI.  
**Aceptación**: perfil consistente y persistencia mínima de respuestas (con consentimiento).

## Tarea 3 — Generación de carteras (regla simple) + explicabilidad
**Objetivo**: crear **3 alternativas** por perfil, con límites básicos.  
**Funcional**: plantillas por perfil (bandas RF/RV/monetarios), límites por fondo/gestora; evitar clases con fee de entrada y TER alto según citas.  
**Técnico**: selección basada en fragmentos recuperados (sin BBDD); función de reparto de pesos con *rounding* a 1%.  
**Aceptación**: 3 carteras válidas, cada una con **≥3 citas** (riesgo, costes, política) y límites respetados.

## Tarea 4 — PDF & Email
**Objetivo**: entregar un documento claro y trazable.  
**Funcional**: portada, perfil, 3 carteras, “por qué”, citas, riesgos clave, disclaimers.  
**Técnico**: motor de plantillas PDF; envío por email; registro de envío.  
**Aceptación**: PDF < 2 MB, render uniforme y enlaces/citas legibles.

## Tarea 5 — Observabilidad mínima y legales
**Objetivo**: visibilidad y control.  
**Funcional**: panel simple con nº de documentos indexados, % propuestas con ≥3 citas, tiempos medios.  
**Técnico**: logs estructurados; almacenamiento UE; disclaimers en UI/PDF.  
**Aceptación**: panel visible, métricas actualizadas y disclaimers verificados.

---

# 13) Criterios de aceptación del MVP
- Propuesta en < **10 s** con **3 carteras** y **≥3 citas** por propuesta.  
- PDF generado y enviado por email correctamente.  
- Trazabilidad: lista de documentos citados con `[tipo, página, fecha_doc, URL]`.  
- Disclaimers visibles y consentimiento registrado.

---

# 14) Anexo (pendiente de definir en posteriores iteraciones)
- Umbrales por categoría (TER máximos orientativos y antigüedad).  
- Señales macro mínimas para ajuste ligero.  
- Texto final de disclaimers legales (formato jurídico).  
- Golden set de pruebas (perfiles y fondos representativos).
