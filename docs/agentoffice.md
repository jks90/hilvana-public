# 🏢 AgentOffice — instalarlo y conectarlo a Hilvana

**AgentOffice** es una oficina de agentes de IA (Claude Code o Codex) que trabaja en tus repositorios:
un PO reparte un objetivo en tareas, los agentes de back, front, QA y documentación las hacen —cada
una en su rama—, un revisor las revisa y, si pasan, se fusionan. Lo ves en directo dentro de Hilvana
(**Config ▸ Plugins ▸ Agentes**) y sus QA usan Hilvana para verificar las APIs con flows.

> **Requiere** Hilvana **Pro o Business** (el plugin está en Pro) y una suscripción de **Claude**
> (Pro/Max) y/o **ChatGPT** (para Codex): los agentes gastan la cuota de esas suscripciones.

## Cómo encaja

```
   tu navegador ──► Hilvana (Docker, :9998) ──/agents/──► AgentOffice (:7420)
                                                              │
                                                              └─► claude / codex trabajan en tus repos
```

AgentOffice se distribuye **compilado y firmado** (sin código fuente) y se descarga **desde tu Hilvana**,
que lo empareja con él. Hay dos formas de instalarlo; Hilvana te ofrece las dos en
**Config ▸ Plugins ▸ Agentes · AgentOffice ▸ Instalar**:

| | 💻 **En este PC** (recomendada) | 🖥 **En un servidor** (Docker) |
|---|---|---|
| Para quién | Trabajas en tu ordenador | Servidor sin escritorio (o Windows/macOS, por ahora) |
| Escritorio: burbuja de chat, modo Blender, control de teclado/ratón | ✓ | ✗ |
| Tus herramientas (git-lfs, compiladores, USB…) y tu sesión de `claude`/`codex` | ✓ las del PC | solo lo que trae la imagen; login dentro del contenedor |
| Cómo se instala | una orden en la terminal | un clic (Hilvana con el socket de Docker) |
| Se actualiza | solo, desde Hilvana | ⬆ Actualizar recrea el contenedor |

La instalación de Hilvana debe estar **vinculada** a tu cuenta (Config ▸ Licencia): sin ello no se puede
descargar AgentOffice ni verificarlo.

## 1. En este PC (recomendada · Linux x64/arm64)

En **Plugins ▸ Instalar ▸ 💻 En este PC** Hilvana te da una orden ya emparejada. Pégala en una terminal
de tu PC (en tu sesión, sin `sudo`):

```bash
curl -fsSL http://localhost:9998/plugins/agent-office/native/install.sh | sh
```

El script:

1. baja el **Node oficial** de nodejs.org (LTS 22) y comprueba su sha256,
2. baja el **paquete firmado** de AgentOffice a través de tu Hilvana (su firma se comprueba al arrancar),
3. lo deja en `~/.hilvana/agentoffice/` — `node/`, `app/` (el paquete) y `data/` (tus datos, fuera del
   paquete para que las actualizaciones no los muevan),
4. guarda el par con Hilvana en `agentoffice.env` (0600) y lo registra como **servicio de tu usuario**
   (`hilvana-agentoffice`), que arranca con tu sesión.

No necesitas Node instalado. Sí necesitas **Claude Code** (`claude`) y/o **Codex** con tu sesión iniciada:
AgentOffice usa la de tu PC. Si Hilvana corre en Docker, su contenedor necesita
`--add-host=host.docker.internal:host-gateway` (el panel avisa si falta).

```bash
systemctl --user status hilvana-agentoffice        # estado
journalctl --user -u hilvana-agentoffice -f        # registro
systemctl --user disable --now hilvana-agentoffice && rm -rf ~/.hilvana/agentoffice   # desinstalar
```

Volver a ejecutar la orden **reinstala sin perder datos**. Si el puerto 7420 está ocupado (otro
AgentOffice), el script se para sin tocar nada.

## 2. En un servidor (Docker)

Para instalar con un clic, Hilvana necesita el socket de Docker. Si tu contenedor no lo monta,
recréalo añadiendo `-v /var/run/docker.sock:/var/run/docker.sock`:

```bash
docker run -d --name hilvana --restart unless-stopped \
  -p 9998:3001 -p 1025:1025 \
  -v /ruta/a/tu/workspace:/app/flows \
  -v $HOME/dev:$HOME/dev \
  -v /var/run/docker.sock:/var/run/docker.sock \
  juankanh/hilvana:6.2.1
```

- `-v $HOME/dev:$HOME/dev`: AgentOffice recibe **las mismas carpetas que Hilvana** (salvo el socket),
  también los volúmenes con nombre; si tu workspace usa **enlaces** a los repos (ver paso 5), móntalos en
  la misma ruta.
- Ya no hacen falta `FLOW_AGENTS_URL` ni `FLOW_AGENTS_TOKEN`: los pone Hilvana.

**Plugins ▸ Instalar ▸ 🖥 En un servidor ▸ Instalar con un clic.** Hilvana descarga `juankanh/agentoffice`
(la última versión), crea la red `hilvana-agentoffice`, arranca el contenedor `hilvana-agentoffice` con su
propio token, el volumen `hilvana-agentoffice-data` (datos y sesiones de las CLIs) y tus carpetas, y le dice
dónde está el workspace. La imagen trae git (con git-lfs) y las CLIs de Claude Code y Codex.

**Antes, vincula la instalación a tu cuenta** (Config ▸ Licencia): sin la vinculación AgentOffice no se
verifica y arranca bloqueado. Si no lo está, el botón es **Vincular cuenta e instalar**: abre Licencia y, al
cerrarla ya vinculado, vuelve a Plugins e instala solo. Si ya lo instalaste sin vincular, basta con vincular:
AgentOffice se desbloquea solo en menos de un minuto.

**Sin acceso a Docker** el panel muestra el `docker run` equivalente, con el token ya preparado; después
pulsa **Comprobar**.

## 3. Iniciar sesión en los agentes (solo en Docker, una vez)

Desde el panel de AgentOffice (Ajustes ▸ motores) o en una terminal:

```bash
docker exec -it hilvana-agentoffice claude auth login
docker exec -it hilvana-agentoffice codex login    # si también usas Codex
```

La sesión se guarda en el volumen: sobrevive a reinicios y actualizaciones. Para el **Guía** también
puedes poner una clave de API (Anthropic, OpenAI o IA local) en los Ajustes de AgentOffice.

> **Codex por SSH**: `codex login` redirige el navegador a `127.0.0.1:1455` del servidor. Usa
> `codex login --device-auth` si tu versión lo admite, o un túnel `ssh -L 1455:127.0.0.1:1455`.

## 4. Actualizar

En **Config ▸ Plugins** aparece **⬆ Versión nueva X** cuando hay otra publicada. En tu PC, **Actualizar**
baja el paquete firmado nuevo, lo prueba en un puerto aparte y cambia de versión (si la nueva no arranca,
vuelve a la anterior; tus datos no se mueven). En Docker, recrea el contenedor con la imagen nueva (los
datos y sesiones se quedan en el volumen).

## 5. Proyectos: salen de las carpetas del workspace

Cada **carpeta de primer nivel** del workspace de Hilvana es un **proyecto** de AgentOffice (aunque
todavía no tenga flows). Para que sus agentes trabajen en un repo, enlázalo **dentro** de la carpeta del
proyecto: AgentOffice deduce el repo de cada enlace que apunte dentro de un repositorio git.

```bash
# proyecto «tienda» con su repo de API y su repo web
mkdir -p ~/workspace/tienda
ln -s ~/dev/tienda-api/flows ~/workspace/tienda/api     # repo «api» (los flows viven en el repo)
ln -s ~/dev/tienda-web       ~/workspace/tienda/web     # repo «web»
ln -s ~/docs/tienda          ~/workspace/tienda/docs    # documentación (no es un repo: solo se lee)
```

En el panel **Proyecto** de Hilvana cada enlace muestra a dónde apunta (`→ ~/dev/tienda-api/flows`).
Las carpetas que empiezan por `.` o `_`, `assets/` y `privado/` no son proyectos.

## 6. Avisos al móvil (opcional)

Para enterarte de lo que te necesita (tarea esperando revisión, una pregunta de un agente, sin cuota,
un atasco, un fallo) por **Telegram**, crea `telegram.json` en la carpeta de datos de AgentOffice (en tu PC, `~/.hilvana/agentoffice/data/`; en Docker, `/data/` del contenedor):

```json
{ "enabled": true, "botToken": "123456:ABC…", "chatIds": ["tu_chat_id"] }
```

Prueba: `curl -X POST localhost:7420/api/telegram/test`.

## 7. Revisión de las tareas

Por proyecto (Ajustes ▸ Proyecto) o para todos:

| Política | Qué pasa al terminar una tarea |
|----------|-------------------------------|
| **manual** (por defecto) | espera a que la apruebes o la devuelvas tú |
| **auto-qa** | un revisor IA la revisa: aprueba y fusiona lo que está bien, devuelve solo lo que bloquea; lo que no puede comprobar (levantar servicios, e2e contra un servidor) lo deja anotado; tras 3 devoluciones, te la deja a ti |
| **auto** | se aprueba sola si pasan las verificaciones que declare la tarea |

Antes de revisar, la rama se pone al día con la principal; los choques en ficheros de historial
(`CHANGELOG.md`…) se combinan solos. Lo que toque ficheros sensibles (Dockerfile, `package.json`,
workflows) siempre te espera a ti.

## 8. Qué pueden tocar los agentes

Los agentes trabajan en una **copia aislada de cada repo** (git worktree, una rama por tarea) y no
hacen `push`. Aun así, ejecutan comandos con **tu usuario**: pueden leer otros ficheros de la máquina
y, el QA, los flows de todo el workspace por el MCP de Hilvana. Úsalo en una máquina y un usuario en
los que confíes, y revisa lo que se fusiona.

## 9. Marketplace: compartir roles, skills y agentes

La pestaña **🛒 Marketplace** de AgentOffice sirve para compartir lo que has montado (roles, skills y
agentes) con tu organización de Hilvana o con todo el mundo, y para instalar lo que han publicado
otros.

**Requisitos:** que Hilvana esté **vinculado a tu cuenta cloud** (Config ▸ Licencia) y que
AgentOffice esté instalado desde Hilvana (pasos 1 o 2: el token del par lo pone Hilvana). AgentOffice nunca habla con la
nube directamente: pasa por Hilvana (`/account-link/marketplace/*`), y el token de vinculación no
sale de Hilvana.

### Las tres pestañas

| Pestaña | Qué ves |
|---------|---------|
| **🔒 Privado** | Lo que tiene tu empresa **en esta instalación**, aún sin compartir. Dos secciones: **🤖 Agentes en el banquillo** (los que no están fichados en el proyecto elegido arriba; primero 🟢 Disponibles y luego un grupo por proyecto) y **🎭 Roles** (por carpeta del catálogo: primero las que usa la plantilla del proyecto, al final ⭐ De serie). Desde aquí se ficha, se edita, se despide y se publica. |
| **👥 Mi team** | Lo publicado por tu organización. Solo lo ven sus miembros. Puede llevar memoria. |
| **🌍 Público** | Lo publicado por todas las organizaciones y **ya aprobado por moderación**. Nunca lleva memoria. |

Las tres comparten barra: buscar, filtrar por tipo (🎭 rol, 🧩 skill, 🤖 agente) y ↻. Los grupos de
Privado se pliegan con ▸ y recuerdan cómo los dejaste; al buscar se abren todos. Al pulsar la etiqueta
de rol de un agente, en cualquier pantalla, se abre Privado con ese rol resaltado.

> Desde octubre de 2026 el banquillo y los roles ya no aparecen en **Agentes**: están en Marketplace ▸ 🔒 Privado.

### Qué se puede publicar y dónde

Cada tarjeta lleva **⬆ Publicar**. El diálogo pide ámbito, versión (semver, `1.0.0` por defecto) y un
resumen, y enseña **qué ficheros se van a subir** antes de enviar nada.

| Qué | 👥 Mi team | 🌍 Público | Qué viaja |
|-----|:---:|:---:|-----------|
| **Rol** (de tu catálogo, de un repo `.claude/agents/*.md` o **de serie**, como el Coordinador) | ✅ | ✅ (con moderación) | Solo el `.md` del rol: prompt, tipo, modelo y la lista de skills/MCP que usa |
| **Skill** (Agentes ▸ Skills ▸ ⬆ Publicar) | ✅ | ✅ (con moderación) | La carpeta de la skill (`SKILL.md` + sus ficheros; si trae código se marca) |
| **Agente** (con su estado y memoria) | ✅ | ❌ | Su rol, motor y modelo, y su **memoria** (y la del proyecto) |

- **Público pasa por moderación:** queda **⏳ pendiente de moderación** hasta que el equipo de
  Hilvana lo aprueba. Hasta entonces solo lo ve tu organización. En Mi team se publica al momento.
- **Lo que nunca sale:** en público nunca va memoria. Tus rutas personales se sustituyen por
  `{{HOME}}` y, si se detecta un posible secreto (token, contraseña, clave), **la publicación se
  rechaza** sin subir nada.
- **Las skills de un rol no viajan con él.** Si tu rol usa skills propias, publícalas también, porque
  si no quien lo instale no las tendrá.
- **Roles de serie:** se publican como un `.md` generado a partir de su definición. Al instalarlos
  donde ya existen de serie no cambia nada, porque el de serie tiene prioridad. Lo que hace de verdad
  el Coordinador (revisar, aprobar o devolver tareas) va en el código de AgentOffice, no en el prompt.
- **Autor:** aparece el nombre configurado en AgentOffice (`authorName`, por `POST /api/settings`) o,
  si no hay ninguno, tu usuario del sistema.

### Instalar

**⬇ Instalar** en una tarjeta de Mi team o Público:

- **Rol:** se guarda en el catálogo, en `roles/marketplace/<organización>/<nombre>.md`, y aparece
  en Privado ▸ Roles.
- **Skill:** se instala en el catálogo de skills. Si **trae código ejecutable**, pide confirmarlo
  (🛡) antes de escribir nada.
- **Agente:** se contrata en el banquillo y pide en **qué proyecto guardar su memoria**.
- Si ya existe algo con ese nombre, pide confirmación para sobrescribirlo. Las tarjetas dicen si ya
  lo tienes instalado (✓) o si hay una **versión nueva** (⟳ Actualizar).

## Problemas frecuentes

| Ves | Causa y solución |
|-----|------------------|
| «No hay un AgentOffice en http://host.docker.internal:7420 (fetch failed)» | AgentOffice no está arrancado (`systemctl --user status hilvana-agentoffice`) o falta `--add-host=host.docker.internal:host-gateway` en el contenedor de Hilvana |
| «AgentOffice pide token» | el par con Hilvana no coincide: vuelve a ejecutar la orden de instalación (en tu PC) o **Reinstalar** (Docker); conserva los datos |
| La oficina abre pero no sale ningún proyecto | el workspace no tiene carpetas de primer nivel, o el contenedor no monta las rutas de los enlaces |
| `curl … install.sh | sh` dice «El puerto 7420 ya está en uso» | hay otro AgentOffice (el contenedor o una instalación antigua): páralo (`docker stop hilvana-agentoffice`) y repite la orden |
| Una tarea vuelve una y otra vez «Tu rama choca con…» | el agente no consigue resolver el choque: a los 3 intentos se queda esperando que lo mires tú |
| Página en blanco al abrir Hilvana por `http://<IP>` | actualiza Hilvana a 5.21.3 o posterior |
| Los agentes no hacen nada y la oficina dice «sin cuota» | se acabó la ventana de tu suscripción de Claude/Codex: reanudan solos al reiniciarse |
| El Marketplace dice «Esta instalación no está vinculada…» y Hilvana sí lo está | AgentOffice no manda su token a Hilvana (versiones anteriores a la del 9-oct-2026): actualiza AgentOffice y reinstálalo desde Hilvana para rehacer el par |
| «Hilvana rechaza a AgentOffice…» en el Marketplace | el par con Hilvana no coincide: reinstálalo desde Hilvana (conserva los datos) |
| El Guía se queda en «trabajando…» o un botón (Sí de una 🛡) no responde | versiones anteriores a Hilvana 5.26: el navegador tenía agotadas sus 6 conexiones con Hilvana por tener varias ventanas abiertas. Actualiza a 5.26 (las conexiones en vivo van por WebSocket); mientras, cierra otras ventanas de Hilvana y pulsa ■ Parar. El Guía avisa a los 10 s si un mensaje no sale |
| El navegador del agente abre una ventana aparte | AgentOffice anterior a oct-2026: ahora va oculto y se ve en 🌐 Navegador (`AO_BROWSER_HEADLESS=0` para la ventana) |
| Un rol publicado en Público no aparece | está **⏳ pendiente de moderación**: hasta que se aprueba solo lo ve tu organización |

AgentOffice es un producto privado: solo se distribuye compilado y firmado a través de Hilvana.
