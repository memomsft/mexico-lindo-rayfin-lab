# Prerrequisitos — Laboratorio Fabric Apps (Rayfin) "México Lindo"

Checklist a completar **antes** del evento. Cada punto es verificable por el
participante sin depender del facilitador.

## 1. Acceso y licencias

- [ ] Cuenta de Microsoft con acceso a Microsoft Fabric. Se recomienda un capacity F8 para el area de trabajo
- [ ] Licencia de Power BI Pro para poder crear y publicar modelos semanticos.
- [ ] Licencia de **GitHub Copilot** activa (individual o de negocio) para cada participante que hará el ejercicio hands-on.
- [ ] Permisos de **Workspsce Admin** en el workspace de Fabric que se usará para el ejercicio.

## 2. Configuración de administrador de Fabric (requiere un Tenant Admin)

- [ ] Workload **"Fabric Apps (preview)"** habilitado en *Fabric Admin Portal → Tenant settings* (para toda la organización o
      para el grupo de seguridad que participará).
- [ ] Setting **"Semantic Model Execute Queries REST API"** habilitado en *Fabric Admin Portal → Integration settings*. Este permiso es el que
      usa la app para consultar el modelo semántico vía DAX.

> Si quien coordina este ejercicio en tu organización no es Tenant Admin, este punto
> debe delegarse a su equipo de TI/gobierno de datos con **al menos unos días
> de anticipación** — es la causa más común de bloqueo el día del evento.

## 3. Workspace y capacidad

- [ ] Workspace dedicado para el ejercicio (ej. `WS-MexicoLindo-Lab`).
- [ ] Capacidad Fabric asignada al workspace, en una región donde **Fabric
      App (preview)** esté disponible. Válidas para este ejercicio:
      `North Central US`, `West US`, `West US 2`, `Central US`, `Francia Central`.
      **México Central no es válida** para este ejercicio
      (Fabric App no disponible ahí a la fecha de esta guía).

## 4. Entorno local de cada participante

- [ ] **Node.js** (LTS) y **npm** instalados.
- [ ] Terminal con acceso a internet saliente (npm, GitHub).
- [ ] **Azure CLI** instalado en la terminal - `winget install --exact --id Microsoft.AzureCLI`
- [ ] Git instalado, y el repo de este ejercicio clonado localmente.

### 4.1 GitHub Copilot en VS Code (obligatorio para el ejercicio)

Todo el laboratorio se hace con *vibe development* usando GitHub Copilot, así que
cada participante necesita tenerlo funcionando **antes** del evento:

- [ ] **VS Code** instalado (versión reciente) —
      `winget install --exact --id Microsoft.VisualStudioCode`
- [ ] Extensión **GitHub Copilot** instalada en VS Code
      (ID: `GitHub.copilot`).
- [ ] Extensión **GitHub Copilot Chat** instalada en VS Code
      (ID: `GitHub.copilot-chat`) — es la que habilita el chat y el
      **modo agente**.
- [ ] **Sesión iniciada** en VS Code con la cuenta de GitHub que tiene la
      licencia de Copilot activa (icono de Copilot en la barra inferior →
      *Sign in*). Verifica que NO aparezca "Copilot: not signed in".
- [ ] **Licencia de GitHub Copilot activa** para esa cuenta (individual,
      Business o Enterprise) — sin licencia, la extensión se instala pero no
      responde.
- [ ] **Modo agente de Copilot habilitado** en VS Code: abre el panel de
      Copilot Chat y confirma que puedes seleccionar **Agent** en el selector
      de modo (Ask / Edit / **Agent**). Si no aparece, actualiza VS Code y las
      extensiones a la última versión.
- [ ] (Recomendado) Política de organización que **permita** GitHub Copilot y
      su acceso a los repos — en tenants corporativos, un admin puede tener
      bloqueado Copilot o el modo agente. Verifícalo con anticipación.

> **Alternativa por terminal:** quien prefiera línea de comandos puede usar el
> **GitHub Copilot CLI** (`copilot`) en vez de VS Code. Para este laboratorio
> guiado recomendamos VS Code + modo agente, porque puede leer el repo completo
> y conducir el paso a paso.

### 4.2 Laboratorio guiado por Copilot (opcional pero recomendado)

Este repo incluye un archivo **`.github/copilot-instructions.md`** que le enseña
a Copilot a conducir el laboratorio como tutor: presenta **un paso a la vez**,
explica qué se hace y por qué, y solo avanza cuando tú confirmas. Para usarlo:

- [ ] Abre la **carpeta del repo clonado** en VS Code (no un archivo suelto —
      la carpeta, para que Copilot detecte `.github/copilot-instructions.md`).
- [ ] Abre **Copilot Chat en modo Agent**.
- [ ] Escribe algo como: *"Vamos a hacer el laboratorio México Lindo, guíame
      paso a paso"*. Copilot leerá los docs del repo y empezará a conducirte.

> Copilot **guía y valida** los pasos de portal (Foundry, Fabric, Azure) — no
> hace clic por ti — y **puede ejecutar** los pasos locales (git, terminal,
> archivos) confirmando antes.

## 5. Verificación rápida (el día antes del evento)

- [ ] Cada participante puede iniciar sesión en https://app.fabric.microsoft.com
- [ ] Cada participante ve el workspace del ejercicio en su panel izquierdo.
- [ ] Cada participante puede crear un ítem de prueba tipo **"App"** en ese
      workspace (si no aparece la opción, el workload no está habilitado o el
      workspace está en una región no soportada — ver punto 2 y 3).
- [ ] `npx --version` y `node --version` corren sin error en la terminal de
      cada participante.
- [ ] `az login` corre sin error en la terminal de
      cada participante.
- [ ] En VS Code, el icono de **Copilot** muestra sesión iniciada (no "not
      signed in") y el panel de **Copilot Chat** permite seleccionar el modo
      **Agent**.

## 6. Solo si vas a hacer la Parte 2 (Foundry IQ) — ver `PARTE2-FOUNDRY-IQ.md`

- [ ] Proyecto de **Microsoft Foundry** creado (toggle "New Foundry" activado).
- [ ] **Foundry IQ resource** creado con anticipación (Knowledge → Knowledge
      bases → "Create new resource"). Es un paso obligatorio: sin este
      recurso no se puede crear el Knowledge Base.
      - La región puede fallar por falta de capacidad ("This region is at
        capacity, Azure AI Search isn't accepting new resources in the
        selected region") — ten una región alternativa lista antes del
        evento.
      - Elige **Basic** como pricing tier (el default puede aparecer
        preseleccionado como Standard S1) — es el mínimo requerido para que
        agentic retrieval funcione, y suficiente para este ejercicio.
- [ ] Rol **Workspace Admin** en el workspace de Fabric (`mexico-lindo-ws`)
      — necesario para poder agregar el Foundry IQ resource a "Manage
      access". Contributor (el mínimo de la Parte 1) no alcanza. Distinto
      del **Tenant Admin (Fabric Admin)** que configuró los tenant settings
      al inicio de este documento — ese es a nivel de todo el tenant, este
      es acotado a este workspace.
- [ ] Un **modelo de embeddings desplegado** en Azure OpenAI/Foundry Models
      (`text-embedding-3-small` alcanza) — lo pide la pantalla de
      configuración de la fuente OneLake, no es opcional.
- [ ] Un **modelo de completions (chat) desplegado** para el agente
      (`gpt-5` en este laboratorio) — es el que razona sobre cada
      transcripción al clasificar el sentimiento; se selecciona al crear el
      Foundry Agent. Distinto del embedding model.
- [ ] Dos **asignaciones de rol RBAC** que NO se crean solas (ver detalle en
      `PARTE2-FOUNDRY-IQ.md` → "Dos roles de fricción"):
- **Cognitive Services User** para la managed identity del **Search**
        sobre el recurso de AI — sin esto el indexer indexa 0/30.
      
- **Search Index Data Reader** para la managed identity del **Foundry
  project** sobre el Search — sin esto el agente da `403` al conectar
        el KB.  
- El Azure AI Search debe estar en **API access control = Role-based**
        (o Both) para que estas identidades funcionen.


---

*Nota: este checklist cubre únicamente lo necesario para correr el ejercicio
con datos sintéticos. No sustituye una evaluación de gobierno de datos ni de
arquitectura para cargas productivas reales.*

