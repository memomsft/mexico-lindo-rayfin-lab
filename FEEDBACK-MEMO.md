# Feedback para Memo — Lab México Lindo (Foundry IQ + Foundry Agent)

> **De:** Gabriela Drumond · **Para:** Memo
> **Contexto:** Completé la **Parte 2** del lab (Foundry Agent para sentiment
> analysis de transcripciones). Este documento resume todo lo que necesito
> alinear contigo antes de que lo publiquemos / lo demos al equipo.

---

## TL;DR — lo más importante

1. **Dos roles RBAC de fricción** que NO se asignan solos al recrear el
   entorno desde cero. Sin ellos el lab se rompe en dos puntos distintos.
   👉 Es el feedback #1 — hay que documentarlos como prerrequisito explícito.
2. **Falta el modelo `gpt-5`** (completion) en la lista de prerrequisitos.
3. **La Parte 2 ya está completa** (antes era placeholder). Resultado:
   **96,7% de exactitud** (29/30). Ya actualicé los docs del repo.
4. Hay una decisión de diseño clave — la **regla anti-cola** — que conviene
   dejar explícita para que el agente no "haga trampa" con el orden del dataset.

---

## 1. Los dos roles RBAC de fricción (crítico)

El entorno usa **Role-based access control** (no keys) tanto en el Azure AI
Search como en el recurso de Foundry. Al recrear todo desde cero, **ninguno de
estos dos roles se asigna automáticamente**, y cada uno rompe una fase distinta:

| # | Rol a asignar | Identidad (quién) | Recurso destino (dónde) | Síntoma si falta |
|---|---|---|---|---|
| 1 | **Cognitive Services User** | Managed identity del **Azure AI Search** (`<nombre-del-search>`) | Recurso de AI **`<recurso-de-ai>`** | Indexer falla con **PermissionDenied → 0/30** documentos indexados (Fase 1) |
| 2 | **Search Index Data Reader** | Managed identity del **proyecto de Foundry** | Azure AI Search **`<nombre-del-search>`** | El agente devuelve **403 Forbidden** al conectar el Knowledge Base (Fase 2) |

**Puntos a alinear contigo:**

- Ambos se asignan vía **IAM → Add role assignment → Managed identity**.
- Hay que esperar **~2-3 min de propagación** después de asignar cada uno
  (me pasó de ver 10/30 parcial justo después de asignar — era propagación).
- Sugerencia: agregarlos como **checklist obligatorio en `PRERREQUISITOS.md`**
  (ya dejé la sección puesta, pero quiero que valides los nombres exactos de
  las identidades en tu tenant, por si difieren del mío).

---

## 2. Detalle del troubleshooting del indexer (por si lo documentamos)

Además del rol #1, durante la Fase 1 pasé por esta secuencia — vale la pena que
la conozcas porque un peer nuevo probablemente la repita:

1. **0/30 PermissionDenied** → se resolvió asignando *Cognitive Services User*.
2. **10/30 parcial** → NO era un error nuevo; era **propagación de RBAC**.
   Esperar y re-ejecutar el indexer lo llevó a avanzar.
3. **0/0 (change-tracking)** → el indexer creía que "no había nada nuevo".
   Se resolvió con **Reset + Run** en el indexer → llegó a **30/30**.

👉 Recomendación: documentar el **Reset + Run** como paso de recuperación
estándar, porque el mensaje "0/0" es confuso (parece error, es cache).

---

## 3. Prerrequisito faltante: modelo `gpt-5`

La guía original listaba solo el modelo de embeddings
(`text-embedding-3-small`). Para la Parte 2 hace falta también un modelo de
**completion desplegado** — usé **`gpt-5`** (Global Standard, v2025-08-07).

- Ya lo agregué a `PRERREQUISITOS.md` y a `PARTE2-FOUNDRY-IQ.md`.
- **A validar contigo:** ¿quieres fijar `gpt-5` como el modelo oficial del lab,
  o dejarlo como "gpt-5 o equivalente de completion" para más flexibilidad?

---

## 4. Regla anti-cola (decisión de diseño a alinear)

El dataset de 30 transcripciones está **ordenado por categoría**
(`call-001..015` positivo, `016..025` neutral, `026..030` negativo, según el
campo `sentimiento_referencia`). Esto crea un riesgo: un agente "perezoso"
podría **inferir el sentimiento por el número de ID** en vez de leer el
contenido real de la llamada.

- Por eso el prompt incluye una **regla anti-cola** explícita: *clasificar
  solo por el contenido de la transcripción, nunca por el ID ni el orden*.
- **La prueba de que funcionó:** el único error del agente fue `call-030`
  (ground truth = negativo, predijo neutral). Si estuviera "haciendo trampa"
  por orden de ID, habría acertado ese por posición. El hecho de que falle
  ahí **demuestra que leyó el contenido**, no el orden. 💡

👉 A alinear: ¿mantenemos el dataset ordenado (bueno para explicar la trampa)
o lo mezclamos (más realista)? Yo voto por **mantenerlo ordenado + documentar
la regla anti-cola** como lección de diseño de agentes.

---

## 5. Resultado de la evaluación (Parte 2)

Corrí el agente sobre las 30 transcripciones y comparé contra el ground truth:

- **Exactitud global: 96,7% (29/30)**

| Categoría | Aciertos | Exactitud |
|---|---|---|
| Positivo | 15/15 | 100% |
| Neutral  | 10/10 | 100% |
| Negativo | 4/5   | 80% |

- Único error: `call-030` (negativo → clasificado como neutral).

---

## 6. Decisiones de diseño ya tomadas (para tu visto bueno)

| Tema | Decisión que tomé | ¿OK contigo? |
|---|---|---|
| Idioma de salida del agente | **Español** | ⬜ |
| Categorías de sentimiento | **Positivo / Neutral / Negativo** | ⬜ |
| Destino de los resultados | **Dashboard en Power BI / Fabric** | ⬜ |
| Rol del Copilot | Distinguir **dev-time** (construir el agente) vs **run-time** (consumir resultados) | ⬜ |
| Refinamiento de prompt | No hice tuning adicional — el prompt base + anti-cola bastó | ⬜ |
| Región | **West US 3** (misma que la Parte 1) | ⬜ |

---

## 7. Qué ya actualicé en tu repo

Para que sepas exactamente qué toqué (solo **agregué**; no borré contenido
tuyo — solo quité las marcas de *"placeholder / por completar"*):

- ✅ **`docs/PARTE2-FOUNDRY-IQ.md`** — quité `(placeholder)`, agregué Pasos 4-9
  (crear agente, conectar KB, smoke test, prompt de sentimiento con anti-cola,
  evaluación 96,7% + matriz de confusión, explotación + Copilot) y la sección
  *"Dos roles de fricción"*. Agregué el diagrama de flujo de la Parte 2.
- ✅ **`docs/PRERREQUISITOS.md`** — agregué `gpt-5` + checklist de los 2 roles RBAC.
- ✅ **`README.md`** — quité el placeholder de la Parte 2, agregué el resultado
  96,7% y actualicé la estructura del repo.
- ✅ **`assets/flujo-parte2.svg`** — nuevo diagrama de flujo (mismo estilo visual
  que tu `flujo-ejercicio.svg`).

---

## 8. Preguntas abiertas para ti

1. ¿Los nombres de las **managed identities** (Search y proyecto Foundry) son
   iguales en tu tenant o hay que parametrizarlos en el doc?
2. ¿Fijamos **`gpt-5`** oficial o lo dejamos como "modelo de completion"?
3. ¿**Dataset ordenado** (didáctico) o **mezclado** (realista)?
4. ¿Quieres que yo prepare el **dashboard de Power BI** como Parte 3, o lo
   dejamos como "próximo paso" documentado?
5. ¿Publicamos como está o quieres una **revisión conjunta** antes del push?

---

*Documento preparado por Gabriela Drumond — 17/jul.*
