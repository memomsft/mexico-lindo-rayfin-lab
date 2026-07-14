# Laboratorio: Fabric Apps (Rayfin) + GitHub Copilot — "México Lindo"

Ejercicio hands-on para construir, con lenguaje natural (GitHub Copilot), una
aplicación en **Microsoft Fabric Apps** que se conecta a un **modelo semántico**
publicado en Fabric. Temática: cadena ficticia de restaurantes **México Lindo**.

> ⚠️ **Nota de vigencia:** Fabric Apps y Rayfin están en **preview pública**
> (anunciados en Microsoft Build, junio 2026). Comandos, límites y comportamiento
> pueden cambiar antes de la disponibilidad general (GA). Esta guía está validada
> contra Microsoft Learn (páginas `fabric/apps/*`, actualizadas el 2 de junio de
> 2026) al 10 de julio de 2026. Antes del evento, vuelve a revisar los enlaces de
> la sección [Fuentes oficiales](#fuentes-oficiales) por si hubo cambios.

## Objetivo del ejercicio

Que el participante, siguiendo este repo en vivo durante el evento, construya de
principio a fin:

1. Un dataset cargado en un Lakehouse de Fabric.
2. Un modelo semántico sobre ese dataset.
3. Una Fabric App (plantilla **Data App**) que un agente de código
   (GitHub Copilot) conecta al modelo semántico usando lenguaje natural,
   generando visuales (KPIs, gráficas, tabla) sin escribir DAX ni código de
   autenticación a mano.
4. El despliegue de esa app en el propio tenant de Fabric de tu organización.

**Propósito de la app:** analítico y de monitoreo — un dashboard de solo
lectura sobre el modelo semántico (KPIs, tendencias, ranking de platillos).
**No es una app transaccional**: no captura pedidos ni escribe datos de vuelta
a Fabric. Si más adelante se quiere explorar writeback, es una fase aparte,
no parte de este ejercicio.

**Duración objetivo del taller práctico: 2 horas.** Ver [Agenda y checkpoints](#agenda-y-checkpoints-2-horas).

---

## Requisitos previos

Ver el detalle completo, verificable, en [`PRERREQUISITOS.md`](./PRERREQUISITOS.md).
Resumen rápido:

- Tenant de Fabric con el workload **Fabric Apps (preview)** habilitado por un
  administrador.
- Setting de administrador **"Semantic Model Execute Queries REST API"**
  habilitado (Fabric Admin Portal → Integration settings).
- Workspace con **capacidad Fabric** asignada, en una **región donde Fabric App
  esté disponible** (ver tabla de regiones abajo). Mexico Central y Spain
  Central **no** son válidas para este ejercicio.
- Permisos de **Contributor o Admin** en el workspace.
- Node.js + npm instalados localmente.
- VS Code con la extensión de GitHub Copilot, o el CLI `copilot` instalado
  (terminal).
- Licencia de GitHub Copilot activa.

### Regiones válidas para este laboratorio

Fabric Apps (preview) **no** está disponible en todas las regiones. Regiones
verificadas como disponibles (fuente: Microsoft Learn, región availability,
actualizado 19 de junio de 2026):

| Región | Fabric App disponible |
|---|---|
| US - North Central US | ✅ |
| US - West US | ✅ |
| US - West US 2 | ✅ |
| US - Central US | ✅ |
| Francia Central | ✅ |
| **México Central** | ❌ No disponible |
| **España Central** | ❌ No disponible |

👉 Para el evento, el workspace de práctica debe crearse con una capacidad
asignada en una de las regiones marcadas ✅ (recomendado: **North Central US**
o **West US 2** por latencia razonable desde LATAM). Esto es solo para el
ejercicio con datos sintéticos; no aplica para arquitecturas productivas de
clientes reales, donde la región debe decidirse por residencia de datos y
gobierno, no solo por disponibilidad de la feature.

---

## Flujo del ejercicio

```
CSV sintéticos (este repo)
        │
        ▼
Lakehouse en Fabric (Tablas Delta)
        │
        ▼
Modelo semántico (Power BI / Direct Lake sobre Lakehouse)
        │
        ▼
Fabric App — plantilla "Data App" (Rayfin CLI)
        │  (GitHub Copilot conecta la app al modelo semántico
        │   vía share link, genera visuales y DAX)
        ▼
npx rayfin up  →  App desplegada en el workspace de Fabric
```

---

## Agenda y checkpoints (2 horas)

| Bloque | Tiempo | Qué se valida al cerrar el bloque |
|---|---|---|
| 1. Verificación de prerrequisitos | 0:00–0:15 | Login a Fabric OK, workload Fabric Apps habilitado, workspace en región correcta |
| 2. Carga del dataset al Lakehouse | 0:15–0:35 | Las 4 tablas (`categorias`, `sucursales`, `platillos`, `ventas`) visibles como Delta Tables |
| 3. Creación del modelo semántico | 0:35–0:55 | Modelo publicado, relaciones entre tablas creadas, una medida DAX básica probada (ej. `Total Ventas`) |
| 4. Obtener share link del modelo | 0:55–1:05 | Link de tipo `.../modeling/<model-id>/modelView` copiado |
| 5. Scaffold del Fabric App (plantilla `dataapp`) | 1:05–1:20 | Proyecto local creado, `npm run dev` corre sin errores |
| 6. Prompt a Copilot para conectar el modelo y generar visuales | 1:20–1:45 | La app muestra al menos 1 KPI, 1 gráfica y 1 tabla con datos reales del modelo |
| 7. Ajustes de estilo/branding vía Copilot | 1:45–1:55 | Marca "México Lindo" aplicada de forma consistente |
| 8. Despliegue (`npx rayfin up`) y validación en el portal | 1:55–2:00 | App visible y funcional dentro del portal de Fabric |

Si algún participante se atrasa, los bloques 1–4 (dataset → modelo semántico) se
pueden dar **pre-armados** solo para ese caso puntual, empezando desde el
bloque 5. Esta es una decisión de facilitación, no técnica.

---

## Paso a paso detallado

### Bloque 1 — Verificación de prerrequisitos (0:00–0:15)

Sigue el checklist de [`PRERREQUISITOS.md`](./PRERREQUISITOS.md) punto por
punto. No continúes al bloque 2 si algo falla aquí — son los puntos que más
tiempo hacen perder en vivo.

### Bloque 2 — Cargar el dataset al Lakehouse (0:15–0:35)

1. En el workspace, crea un ítem **Lakehouse** (ej. `lh_mexicolindo`).
2. Sube los 4 archivos de `/data` a **Files** del Lakehouse (arrastrar y
   soltar desde el portal, o "Get data" → "Upload files").
3. Usa **"Load to Tables"** sobre cada CSV (clic derecho → *Load to Tables* →
   *New table*) para convertirlos en tablas Delta:
   - `categorias`
   - `sucursales`
   - `platillos`
   - `ventas`
4. Verifica en el **SQL Analytics Endpoint** del Lakehouse que las 4 tablas
   existen y tienen filas (`SELECT COUNT(*) FROM ventas` debe regresar ~10,000).

> Esta parte usa capacidades estándar de Fabric (Lakehouse, Load to Tables),
> no específicas de Rayfin — conocimiento general de la plataforma, no
> requiere validación adicional contra Rayfin.

### Bloque 3 — Crear el modelo semántico (0:35–0:55)

1. Desde el Lakehouse, selecciona **"New semantic model"**.
2. Nómbralo `sm_mexicolindo`.
3. Incluye las 4 tablas.
4. En el editor del modelo, crea las relaciones:
   - `ventas[id_sucursal]` → `sucursales[id_sucursal]`
   - `ventas[id_platillo]` → `platillos[id_platillo]`
   - `platillos[id_categoria]` → `categorias[id_categoria]`
5. Crea una medida base para probar el modelo:
   ```dax
   Total Ventas = SUM(ventas[monto_total])
   ```
6. Guarda y publica el modelo.

### Bloque 4 — Obtener el share link del modelo (0:55–1:05)

1. Abre el modelo semántico publicado en el **Power BI Service**.
2. Copia el link de la barra de navegación — debe tener el formato:
   ```
   https://app.powerbi.com/groups/<workspace-id>/modeling/<model-id>/modelView
   ```
3. Guarda este link, lo vas a pegar en el prompt a Copilot en el bloque 6.

### Bloque 5 — Scaffold del Fabric App con la plantilla `dataapp` (1:05–1:20)

Esta parte sí es específica de Rayfin — validada contra Microsoft Learn
(`fabric/apps/create-app`, `fabric/apps/data-apps-template`).

1. En el workspace, **New item → App**, nómbralo `app-mexicolindo`.
2. En tu terminal local:
   ```bash
   npm create @microsoft/rayfin@latest -- "app-mexicolindo" --template dataapp --workspace <nombre-del-workspace>
   cd app-mexicolindo
   npm run dev
   ```
3. Confirma que el servidor local levanta sin errores (verás una URL local en
   la terminal).

### Bloque 6 — Prompt a Copilot para conectar el modelo semántico (1:20–1:45)

1. Abre VS Code en la carpeta del proyecto y abre el panel de **GitHub
   Copilot Chat** (o ejecuta `copilot` en una terminal).
2. Usa un prompt como el siguiente (ver más ejemplos en
   [`docs/prompts-copilot.md`](./docs/prompts-copilot.md)):

   > Conecta esta app al modelo semántico en
   > `https://app.powerbi.com/groups/<workspace-id>/modeling/<model-id>/modelView`.
   > Quiero un dashboard de ventas de México Lindo con: una tarjeta KPI de
   > ventas totales, una gráfica de barras de ventas por sucursal, una gráfica
   > de línea de ventas por mes, y una tabla con el detalle de platillos más
   > vendidos (nombre, categoría, cantidad vendida, monto total).

3. Copilot generará la conexión al modelo (maneja la autenticación por ti,
   según la documentación oficial) y los componentes visuales.
4. Itera pidiendo ajustes puntuales (ej. "agrega un filtro por sucursal",
   "ordena la tabla por monto total descendente").

> **Limitación conocida (documentada por Microsoft):** una Fabric App
> conectada a un modelo semántico no se puede abrir en su propia ventana de
> navegador fuera del portal de Fabric — el botón "Open" falla en las
> consultas visuales. Para el demo en vivo, la app se muestra **dentro** del
> portal de Fabric, no en una pestaña aparte.

### Bloque 7 — Estilo y marca "México Lindo" (1:45–1:55)

Pide a Copilot ajustes de estilo en una sola instrucción (el template centraliza
el estilo en un archivo, así que un solo prompt actualiza toda la app de forma
consistente):

> Aplica un estilo de marca "México Lindo": paleta de verde, blanco y rojo,
> tipografía moderna sans-serif, esquinas redondeadas en las tarjetas.

### Bloque 8 — Despliegue y validación (1:55–2:00)

```bash
npx rayfin up
```

1. Espera a que termine el despliegue (compila frontend, aplica esquema,
   provisiona/actualiza los child services).
2. Abre el ítem **App** en el portal de Fabric y valida que el dashboard
   carga datos reales del modelo semántico.

---

## Solución de problemas comunes

| Síntoma | Causa probable | Acción |
|---|---|---|
| El workspace no permite crear un ítem "App" | Workload Fabric Apps no habilitado, o workspace en región no soportada | Revisar tenant admin settings y región de la capacidad |
| Copilot no logra conectar al modelo semántico | Falta el permiso "Semantic Model Execute Queries REST API" en el tenant, o el usuario no tiene permisos Build/Read sobre el modelo | Pedir al admin habilitar el setting; verificar permisos del modelo |
| El botón "Open" de la app falla al cargar visuales | Limitación conocida y documentada (preview) para apps conectadas a modelos semánticos | Validar el demo dentro del portal de Fabric, no en ventana aparte |
| `npx rayfin up` falla con error de sesión | Sesión de Rayfin CLI expirada | `npx rayfin login` y reintentar |

---

## Fuentes oficiales

Toda la parte técnica específica de Rayfin/Fabric Apps en esta guía está
validada contra estas páginas de Microsoft Learn (revisadas el 10 de julio de
2026; recordar que la feature está en preview y puede cambiar):

- Overview: https://learn.microsoft.com/en-us/fabric/apps/overview
- Crear tu primera app: https://learn.microsoft.com/en-us/fabric/apps/create-app
- Plantilla Data App (modelo semántico): https://learn.microsoft.com/en-us/fabric/apps/data-apps-template
- Estructura del proyecto: https://learn.microsoft.com/en-us/fabric/apps/project-structure
- Disponibilidad por región: https://learn.microsoft.com/en-us/fabric/admin/region-availability

---

## Estructura de este repo

```
mexico-lindo-rayfin-lab/
├── README.md                  ← esta guía (para el participante)
├── PRERREQUISITOS.md          ← checklist verificable, para el repo y para enviar por correo
├── data/
│   ├── categorias.csv
│   ├── sucursales.csv
│   ├── platillos.csv
│   └── ventas.csv
└── docs/
    └── prompts-copilot.md     ← prompts de ejemplo para el bloque 6
```
