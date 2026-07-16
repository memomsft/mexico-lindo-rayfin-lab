# Prerrequisitos — Laboratorio Fabric Apps (Rayfin) "México Lindo"

Checklist a completar **antes** del evento. Cada punto es verificable por el
participante sin depender del facilitador.

## 1. Acceso y licencias

- [ ] Cuenta de Microsoft con acceso a Microsoft Fabric.
- [ ] Licencia de **GitHub Copilot** activa (individual o de negocio) para
      cada participante que hará el ejercicio hands-on.
- [ ] Permisos de **Contributor o Admin** en el workspace de Fabric que se
      usará para el ejercicio.

## 2. Configuración de administrador de Fabric (requiere un Tenant Admin)

- [ ] Workload **"Fabric Apps (preview)"** habilitado en
      *Fabric Admin Portal → Tenant settings* (para toda la organización o
      para el grupo de seguridad que participará).
- [ ] Setting **"Semantic Model Execute Queries REST API"** habilitado en
      *Fabric Admin Portal → Integration settings*. Este permiso es el que
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
- [ ] **VS Code** instalado, con la extensión de **GitHub Copilot**.
      (Alternativa: CLI `copilot` instalado, para quienes prefieran terminal
      en vez de VS Code).
- [ ] Terminal con acceso a internet saliente (npm, GitHub).
- [ ] **Azure CLI** instalado en la terminal - `winget install --exact --id Microsoft.AzureCLI`
- [ ] Git instalado, y el repo de este ejercicio clonado localmente.

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


---

*Nota: este checklist cubre únicamente lo necesario para correr el ejercicio
con datos sintéticos. No sustituye una evaluación de gobierno de datos ni de
arquitectura para cargas productivas reales.*
