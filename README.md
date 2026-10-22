# Hilvana — Visual API Orchestrator (edición Docker)

**Hilvana** es una herramienta visual + CLI + MCP para componer, ejecutar y verificar
flujos de peticiones HTTP y consultas SQL encadenadas: importas comandos `curl`, conectas
nodos, extraes datos de las respuestas (JSONPath / columnas SQL) y los reutilizas en los
siguientes pasos con `{{variables}}`.

Este repositorio es **solo documentación**: todo lo necesario para usar Hilvana al máximo desde
la **imagen Docker oficial**, sin código fuente.

```
┌─────────────┐   comandos MCP    ┌──────────────────────────────────────┐
│  IA (Claude) ├──────────────────►  contenedor juankanh/hilvana        │
│  claude mcp  ◄──────────────────┤  · Web (canvas en :3001)             │
└─────────────┘  estado/resultados│  · CLI  (flow runner)                │
                                  │  · MCP  (/mcp, 39 tools)             │
       tú miras el canvas ────────►  · SQL  (postgres/mysql/oracle)      │
                                  └──────────────────────────────────────┘
```

## Arranque rápido

```bash
docker run -d \
  --add-host=host.docker.internal:host-gateway \
  -p 9998:3001 \
  -p 1025:1025 \
  --name hilvana \
  juankanh/hilvana:6.0.1
```

- **Web**: http://localhost:9998 — el canvas visual.
- **CLI**: `docker exec flow node cli/run-flow.js --dir flows` — ejecuta baterías de flows.
- **Correo de prueba** 🆕 4.36: **Config → Correo** — SMTP en `localhost:1025` que recibe los correos de tus servicios y los muestra (buzones `@midominiotest.com`, sin cuentas reales); desde la 4.37 la IA lo maneja por MCP (`mail_*`).
- **MCP** (agentes IA): `claude mcp add --transport http hilvana http://localhost:9998/mcp`
  — la IA construye y ejecuta flows **en tu canvas, mientras lo ves**.

## 🗂️ Galería: arranca con 21 flows de verdad

¿Prefieres tocar antes que leer? Monta la **galería de ejemplos** como tu workspace y la app abre
con un tour guiado dentro del propio producto — cada ejemplo se ejecuta con un clic:

```bash
git clone https://github.com/jks90/hilvana-public.git
docker run -d \
  --add-host=host.docker.internal:host-gateway \
  -p 9998:3001 --name hilvana \
  -v "$(pwd)/hilvana-public/examples/workspace:/app/flows" \
  juankanh/hilvana:latest
# abre http://localhost:9998 → icono «Proyecto» → «00 EMPIEZA AQUÍ.md»
```

Dentro: `aprende/` (variables, asserts, entornos, scripts, SQL, data-driven — un concepto por flow,
5 minutos cada uno), `funciones/` (monitor 24x7, webhook desde CI, correo OTP, puente a local),
`hilvana-por-dentro/` (nuestra propia plataforma probada con nuestra propia herramienta: alta,
login, **compra de licencia** y salud — el funnel real, documentado como flow) `paneles/` (los
vistosos para enseñar en una demo) y `scripts/` (🆕 5.14: señales de mercado, semáforo del tiempo y
verificación de pedidos calculados con **scripts JS dentro del flow**, gráficos Mermaid incluidos, y dos flows cuyos
scripts dibujan **botones, páginas y acciones** encima de Hilvana). Todos con asserts, verificados con el CLI antes de publicarse.
Detalle en [examples/README.md](examples/README.md). *(El tour en .md se abre desde la 5.5.0.)*

## Versiones de la imagen

| Versión | Qué trae |
|---------|----------|
| **6.0.0** (recomendada) | 🧵 Versión de mantenimiento: marca Hilvana en el código y la documentación y base para «Iniciar sesión con Google» (aún inactiva). Sin cambios de comportamiento en la app |
| **5.39.0** | 📖 Versión de mantenimiento del manual: prepara la ampliación del [manual](docs/manual/README.md) pantalla a pantalla y su revisión de calidad. Sin cambios de comportamiento en la app |
| **5.38.3** | 🧠 AgentOffice 0.3.3: cada agente con su **recuerdo** (conversaciones a través de todas sus tareas, «que siga con…» desde su ficha, burbuja a demanda), **sesión viva** que se retoma entre tareas y un Blender por lado ([manual 15](docs/manual/15-agentoffice.md)) |
| **5.38.2** | 📖 **Manual pantalla a pantalla** (capítulos 01/09/10/12 ampliados, capítulo [15 AgentOffice](docs/manual/15-agentoffice.md), [PDF](docs/manual/hilvana-manual.pdf)) y AgentOffice 0.3.2: el chat de la burbuja responde a las preguntas con capturas y bocetos, dictado y voz, nueva ronda tras terminar, modo web completo |
| **5.38.1** | 🛠 AgentOffice 0.3.1: la burbuja del agente **se queda al terminar** la tarea (aprobar o devolver con cambios e imágenes desde ella), plegado a solo el muñeco, Blender reabierto tras reiniciar y fusión que aparta sola los cambios locales ([manual 08](docs/manual/08-paneles.md#modos-de-desarrollo-y-chat-con-el-agente-agentoffice--5380)) |
| **5.38.0** | 🛠 **Modos de desarrollo de AgentOffice**: burbuja del agente en un lado de la pantalla toda la tarea, modo Blender (abre el `.blend` del agente en su mitad, cuatro vistas, recarga al guardar) y la burbuja como **chat** con capturas y boceto ([manual 08](docs/manual/08-paneles.md#modos-de-desarrollo-y-chat-con-el-agente-agentoffice--5380)) |
| **5.37.0** | 📱 **App móvil**: adjuntar foto, galería o archivo al crear una tarea de AgentOffice, ficha de cada agente de «Equipo» y atrás que cierra la hoja de detalle ([12](docs/manual/12-movil.md)) (FT-327) |
| **5.36.0** | 📦 **Paquete firmado de AgentOffice verificado de punta a punta** (descarga, firma, arranque y atestación del paquete 0.2.0 publicado) ([08](docs/manual/08-paneles.md#instalar-y-actualizar-agentoffice--525)) y **admin del cloud en el móvil** ([12](docs/manual/12-movil.md)) (FT-313) |
| **5.35.0** | 🤖 **AgentOffice: nuevas opciones de entrega** (carpeta del proyecto accesible, entrega sin commitear, «📥 Llevar a mi checkout», limpieza de ramas vacías) ([08](docs/manual/08-paneles.md#opciones-de-entrega-de-agentoffice--5350)) y la app Android de administración guarda las contraseñas de cada panel ([12](docs/manual/12-movil.md)) (FT-307) |
| **5.34.0** | 📦 **AgentOffice por paquete firmado**: la guía **⬇ Instalar** descarga el paquete firmado a través de tu cuenta vinculada (sin `git clone`), con QA de instalar, actualizar y blindaje ([08](docs/manual/08-paneles.md#instalar-y-actualizar-agentoffice--525)) (FT-302) |
| **5.33.0** | 🏢 La guía **⬇ Instalar** de **AgentOffice** avisa de su **acceso anticipado** (el repositorio es privado) y pliega los pasos para quien ya tiene acceso ([08](docs/manual/08-paneles.md#instalar-y-actualizar-agentoffice--525)) (FT-287) |
| **5.32.0** | 📱 App Android **de administración**: segunda variante (`io.hilvana.mobile.admin`) que se instala junto a la normal y lleva el admin del cloud, Hilvana y AgentOffice por HTTPS ([12](docs/manual/12-movil.md)) (FT-268) |
| **5.31.1** | 🖌️ La **pizarra** vuelve a pintar los dibujos de un flow escrito a mano (por un agente o un script) cuyos elementos no tenían `id`: antes se veían solo en el minimapa |
| **5.31.0** | ➕ **Nuevo proyecto** en AgentOffice: botón siempre visible junto al selector y también en la vista móvil (hoja táctil con nombre y carpeta) ([12](docs/manual/12-movil.md)) |
| **5.30.4** | 📄 La vista móvil `/m` abre también los **`.md`, `.pdf`, scripts y `.csv`** del proyecto; en la app Android, AgentOffice tiene **🎤 dictado** para crear tareas hablando (la IA propone título, descripción y quién la hace), devolver, responder y hablar con el Guía |
| **5.30.3** | 🙈 Las **carpetas ocultas** del panel Proyecto se guardan en el servidor: la vista móvil `/m` y la app Android ocultan las mismas |
| **5.30.2** | 📱 **App Android con AgentOffice**: el botón 🤖 abre AgentOffice en una vista pensada para el móvil (resumen, bandeja con aprobar/devolver, tareas y chat del Guía) y ⚡ vuelve a Hilvana; en la vista móvil `/m` la lista de flows es un **árbol de carpetas** plegable |
| **5.30.1** | ⏹ **Botón Parar** en la barra mientras corre Run Flow o la cadena de un ▶: cancela la petición en curso y no ejecuta lo que falta; Reset también para la ejecución (FT-231) |
| **5.30.0** | 💬 **Chat de cada flow** en la colaboración en vivo (Business): quienes tienen abierto el mismo flow se escriben y la conversación queda guardada con él; en el cloud cada uno aparece con su usuario de login. Los mensajes quedan registrados por seguridad |
| **5.29.0** | 🗒️ **Sticky notes en la pizarra**: notas adhesivas con la fecha de creación y un color a elegir entre 6 pasteles (herramienta `N`, doble clic para editar) |
| **5.28.0** | 📊 **Ver ficheros CSV y TSV**: el panel Proyecto abre los `.csv`/`.tsv` de tu carpeta como tabla (cabecera fija, de 1000 en 1000 filas) o como texto, con separador `,`, `;` o tabulador |
| **5.27.1** | ⚙️ **Ajustes de AgentOffice más cómodos**: el panel ocupa casi toda la ventana, agrupa las secciones (Empresa, Proyecto, Guía y voz, Seguridad), marca las que tienen cambios y deja **Guardar** siempre visible; guardar no cierra el panel y, si cierras con cambios, pregunta. Arregla que «Proyecto» saliera pegada debajo de todas |
| **5.27.0** | 🔑 **Conexión del asistente de IA clara**: Claude y OpenAI muestran el mismo estado (sin configurar, conectado, error, caducado) y el modo activo, con **Probar conexión** (dice el error concreto: clave inválida, caducada, sin red, sin cuota…), **Reconfigurar** y **Desconectar**; el panel de AgentOffice sigue abierto al recargar y el CLI envía el `Content-Type` de `-F 'campo=…;type=…'` |
| **5.26.0** | 🔌 **Tantas ventanas como quieras**: las conexiones en vivo (puente MCP, colaboración y AgentOffice) van por WebSocket, así que con varias ventanas de Hilvana abiertas el Guía y los botones (p. ej. contestar una 🛡) ya no se quedan colgados; si un proxy no admite WebSocket, vuelve sola a la conexión clásica |
| **5.25.1** | 🔌 con varias pestañas de Hilvana abiertas, el Guía de AgentOffice ya no se queda en «trabajando…» (las pestañas en segundo plano liberan conexiones) y el proxy de Agentes corta limpio si AgentOffice se reinicia |
| **5.25.0** | 🧩 **Plugins que se instalan y actualizan desde la app**: Config ▸ Plugins detecta si AgentOffice no está en tu PC (guía con los comandos listos para copiar) y, si va por detrás, lo actualiza con un clic · 🛒 Marketplace de AgentOffice con pestaña 🔒 Privado (banquillo y roles) y roles de serie publicables |
| **5.24.1** | ☁️ el panel Proyecto espera a que despierte tu espacio del cloud en vez de mostrar «El contenedor está despertando» · 🛒 el marketplace de AgentOffice funciona con Hilvana en Docker |
| **5.24.0** | 🔐 **Credenciales en claro bajo control**: al subir un flow al cloud o compartir un enlace, detecta contraseñas y tokens escritos tal cual (SQL, cabeceras, `-u`, cuerpo, URL) y los convierte con un clic a `{{secret:…}}` cifrados · ☁️ el panel Proyecto gestiona carpetas y ficheros del cloud como en local (y el MCP con `source:"cloud"`) |
| **5.23.0** | 🛒 **Marketplace Hilvana**: comparte con tu organización roles, skills y agentes de AgentOffice (con su memoria) o publica especificaciones en el marketplace público (sin memoria, moderado); Hilvana hace de puerta segura hacia tu cuenta · 🧪 tests unitarios del servidor (`npm run test:unit`) |
| **5.22.0** | ⬆️ **Avisa de versiones nuevas** («⬆ x.y.z» junto a Config → novedades y cómo actualizar) y, si arrancas el contenedor con `-v /var/run/docker.sock:/var/run/docker.sock -e FLOW_SELF_UPDATE=on`, **se actualiza con un clic** (y vuelve sola a la anterior si la nueva no arranca) · ⚙️ **Configuración con menú lateral** · 🎤 Ctrl+Espacio habla con el Guía de AgentOffice — [11](docs/manual/11-docker.md#actualizaciones--522) |
| **5.21.3** | 🖥️ Abrir Hilvana por la **IP de un servidor** (`http://192.168.x.x:9998`) ya no deja la página en blanco, y los botones «Copiar» funcionan por HTTP |
| **5.21.2** | 📬 Mejoras internas del correo de prueba (las tools MCP `mail_*` consultan los buzones remotos antes de leer) |
| **5.21.1** | 💾 Botón **Guardar** siempre visible en la barra y el diálogo «Guardar como…» ya no esconde su botón con muchas carpetas (reporte de usuario) — [01](https://github.com/jks90/hilvana-public/blob/main/docs/manual/01-pantalla-principal.md) |
| **5.21.0** | 🗂️ **Panel Proyecto a medida**: los **scripts** (`.js`, `.mjs`, `.ts`, `.py`, `.sh`, `.sql`) se ven y se abren como código, las carpetas **enlazadas** muestran a dónde apuntan, el panel se **ensancha** arrastrando su borde y puedes **ocultar carpetas** — [08](https://github.com/jks90/hilvana-public/blob/main/docs/manual/08-paneles.md#panel-proyecto-a-medida--521) |
| **5.20.2** | 🗑️ El borrado de documentos `.md`/`.pdf` va por su propia ruta (`DELETE /workspace/file`) y la pestaña solo se cierra si el documento era del disco local |
| **5.20.1** | 🗂️ **Ordenar el proyecto desde el panel**: crear carpetas, **arrastrar** flows, documentos y carpetas a otra carpeta, ✏️ mover o renombrar, borrar carpetas vacías y **borrar documentos `.md` y `.pdf`** (un enlace se quita sin tocar el original). Las pestañas abiertas siguen a su fichero — [08](https://github.com/jks90/hilvana-public/blob/main/docs/manual/08-paneles.md#ordenar-el-proyecto--520) |
| **5.19.4** | Incluye lo de las 5.19.0–5.19.3 (no publicadas sueltas): **asserts de errores esperados** (`404`/`4xx` en verde si se cumplen) y con `{{variables}}`, **contexto para el plugin Agentes** (AgentOffice sabe qué flow y nodo miras) y sus **Ajustes tipo Cmd** · 🗂️ El panel Proyecto y `flow_files_list` **ya no repiten los flows** de las copias de trabajo de los agentes (git worktrees dentro de un repo enlazado en el workspace) — [08](https://github.com/jks90/hilvana-public/blob/main/docs/manual/08-paneles.md#carpetas-enlazadas-y-copias-de-los-agentes--5194) |
| **5.19.0–5.19.3** | ✅ **Asserts de errores esperados**: un assert de status `404`/`4xx` deja el nodo en verde si se cumple, y los asserts aceptan `{{variables}}` ([02](https://github.com/jks90/hilvana-public/blob/main/docs/manual/02-nodo-request.md#errores-esperados-y-variables--519)). Plugin Agentes: recibe lo que estás viendo, panel de Ajustes propio y escucha por voz; `flow_open` recarga del disco y `flow_save` no pisa cambios ajenos |
| **5.18.0** | 🧩 **Plugins** (Config ▸ Plugins) con **AgentOffice** como primero: una oficina de agentes de IA que trabaja en tus repos, embebida en Hilvana (Pro). El workspace **sigue enlaces simbólicos**: una carpeta de proyecto puede reunir los `flows/` de varios repos — [08](https://github.com/jks90/hilvana-public/blob/main/docs/manual/08-paneles.md#plugins-y-agentoffice--518) |
| **5.17.0** | 🔔 **Vigilancia en el servidor**: al monitor programado se le añaden **reglas** sobre los datos que extrae el flow (`precio > 2700`, `senal cambia`, `estado contiene ERROR`…). El servidor lo ejecuta cada N min **sin navegador** y avisa por webhook (Slack, Telegram, ntfy, n8n…) **solo cuando una regla se dispara o se recupera**; `notifyUrl` admite `{{secret:X}}`; tool MCP `flow_monitor` (39 tools) — [08](https://github.com/jks90/hilvana-public/blob/main/docs/manual/08-paneles.md) |

Las versiones anteriores (5.12 y antes, desde la 4.x) están en la [tabla completa del manual](docs/manual/11-docker.md#versiones).

```bash
docker pull juankanh/hilvana:6.0.1
```

## Documentación

| Guía | Contenido |
|------|-----------|
| [docs/manual/](docs/manual/README.md) | 📘 **Manual de uso completo con capturas de pantalla anotadas**: la pantalla principal, cada tipo de nodo, variables, **Vista (tamaño de nodos, modo compacto, separación)**, **proyecto (flows/ + Ctrl+S)**, **pizarra**, **correo de prueba (SMTP)**, guía de nodos, modo Live, flow-explore, paneles, CLI, MCP y Docker |
| [docs/instalacion-docker.md](docs/instalacion-docker.md) | Montar la imagen: puertos (web y SMTP de prueba), redes, variables de entorno, persistencia de flows, actualizar de versión, troubleshooting |
| [docs/cli.md](docs/cli.md) | El runner por terminal: flags, baterías, reports, exit codes para CI, nodos SQL y perfiles de conexión |
| [docs/agentoffice.md](docs/agentoffice.md) | 🏢 **AgentOffice**: instalar la oficina de agentes de IA en tu máquina, conectarla a Hilvana, proyectos con enlaces a tus repos, arranque automático, actualizaciones, avisos por Telegram, revisión automática y problemas frecuentes |
| [docs/mcp.md](docs/mcp.md) | Conectar una IA: Claude Code y Claude Desktop, las 39 tools (incl. correo de prueba y vigilancia), seguridad, flujos de trabajo típicos |
| [docs/flows-formato.md](docs/flows-formato.md) | El formato `.flow.json` a fondo: nodos HTTP y SQL, conexiones, variables y extracciones — para escribir flows a mano o con IA |
| [skills/flows/](skills/flows/SKILL.md) | **Skill para agentes IA** (Claude Code): cómo trabajar con Hilvana + [schema de autoría](skills/flows/references/flow-schema.md) — cópiala a tu proyecto |
| [examples/](examples/) | Flows de ejemplo listos para cargar o ejecutar |

## Ficheros listos para usar

| Fichero | Para qué |
|---------|----------|
| [bin/flow-run](bin/flow-run) | **El CLI en tu máquina sin código fuente**: wrapper que ejecuta el runner de la imagen montando tu directorio actual — flows y reports quedan en tu disco. `cp bin/flow-run ~/.local/bin/ && chmod +x ~/.local/bin/flow-run` |
| [docker-compose.example.yml](docker-compose.example.yml) | Compose de referencia: puertos, volumen de flows, env vars del MCP y red de tus APIs |
| [.mcp.json.example](.mcp.json.example) | Config MCP por proyecto para Claude Code (transporte HTTP) |
| [mcp-config.stdio.example.json](mcp-config.stdio.example.json) | Config para clientes MCP **solo-stdio** (vía [`mcp-remote`](https://www.npmjs.com/package/mcp-remote)) |

## Lo esencial en 4 recetas

**1. Ver la web y crear un flow a mano** → abre http://localhost:9998, pulsa *Add Request*,
pega un `curl`, conecta nodos y *Run Flow*.

**2. Ejecutar una batería por terminal (CI)**:

```bash
docker exec flow node cli/run-flow.js --dir flows --report-root /tmp/resumen
docker cp flow:/tmp/resumen ./resumen   # informe completo (report.md, debug por flow…)
```

**3. Que una IA construya el flow mientras lo miras**:

```bash
claude mcp add --transport http hilvana http://localhost:9998/mcp
# abre http://localhost:9998 en el navegador y dile a Claude:
#   «crea un flow que haga login en mi API y liste los usuarios con el token»
```

**4. Guardar lo que la IA construyó y ejecutarlo en CI**: la tool `flow_save` deja el
`.flow.json` en el contenedor; `docker exec flow node cli/run-flow.js --flow flows/mi-flow.flow.json`
lo ejecuta igual que la web.

**5. Trabajar sobre tu carpeta de flows como un proyecto** (4.25): monta `-v /tu/carpeta:/app/flows`
(o `-v /tu/carpeta:/data/flows -e FLOW_FLOWS_DIR=/data/flows`), abre el panel **Proyecto** en la web y
edita/guarda con **Ctrl+S** directamente en tus ficheros — versionables en git y ejecutables por el CLI
sin pasos intermedios.

## Requisitos

- Docker (la imagen es multi-arquitectura estándar `node:20-alpine`).
- Para el MCP: [Claude Code](https://claude.com/claude-code) u otro cliente MCP con
  transporte *Streamable HTTP*.

> La imagen se publica en Docker Hub como
> [`juankanh/hilvana`](https://hub.docker.com/r/juankanh/hilvana).

## Licencia y planes

> Componentes de terceros incluidos en la imagen y sus licencias: [LICENCIAS.md](LICENCIAS.md).

**Hilvana es software propietario** ([`LICENSE`](LICENSE)):

- **Gratis** — uso **personal y no comercial** por una persona, con **todas las funciones**, self-hosted en tu máquina.
- **De pago** — el **uso comercial, en equipo o en la nube** requiere una **licencia de pago**; algunas funciones avanzadas se habilitan con una clave de licencia.

Para una licencia comercial, contacta con el autor a través de este repositorio.

## Activar una licencia

Hilvana funciona **gratis para uso personal**. Para uso comercial, en equipo o en la nube necesitas una licencia (ver [`LICENSE`](LICENSE)). Una vez la tengas:

1. En la app: **Config ▸ Licencia** → pega tu clave → **Activar**. Se guarda en `flows/.license` y se verifica al momento.
2. O por entorno: arranca el contenedor con `-e FLOW_LICENSE="<tu-clave>"`.

La verificación es **offline** (tu clave nunca sale de tu máquina): el servidor comprueba la firma **RS256** con una clave pública empaquetada. Para conseguir una licencia comercial, contacta con el autor a través de este repositorio.
