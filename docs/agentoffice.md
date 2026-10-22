# 🏢 AgentOffice — instalarlo y conectarlo a Hilvana

**AgentOffice** es una oficina de agentes de IA (Claude Code o Codex) que trabaja en tus repositorios:
un PO reparte un objetivo en tareas, los agentes de back, front, QA y documentación las hacen —cada
una en su rama—, un revisor las revisa y, si pasan, se fusionan. Lo ves en directo dentro de Hilvana
(**Config ▸ Plugins ▸ Agentes**) y sus QA usan Hilvana para verificar las APIs con flows.

> **Requiere** Hilvana **Pro o Business** (el plugin está en Pro) y una suscripción de **Claude**
> (Pro/Max) y/o **ChatGPT** (para Codex): los agentes gastan la cuota de esas suscripciones.

## Cómo encaja

```
   tu navegador ──► Hilvana (Docker, :9998) ──/agents/──► AgentOffice (contenedor hilvana-agentoffice, :7420)
                          │                                     │
                          └── mismas carpetas montadas ─────────┤
                                                                └─► claude / codex trabajan en tus repos
```

AgentOffice se distribuye como **imagen Docker** (`juankanh/agentoffice`, código compilado y firmado,
con git y las CLIs de Claude Code y Codex dentro). Hilvana lo instala, lo conecta y lo actualiza.

## 1. Dar a Hilvana acceso a Docker (una vez)

Para instalar con un clic, Hilvana necesita el socket de Docker. Si tu contenedor no lo monta,
recréalo añadiendo `-v /var/run/docker.sock:/var/run/docker.sock`:

```bash
docker run -d --name hilvana --restart unless-stopped \
  -p 9998:3001 -p 1025:1025 \
  -v /ruta/a/tu/workspace:/app/flows \
  -v $HOME/dev:$HOME/dev \
  -v /var/run/docker.sock:/var/run/docker.sock \
  juankanh/hilvana:6.1.0
```

- `-v $HOME/dev:$HOME/dev`: AgentOffice recibe **las mismas carpetas que Hilvana** (salvo el socket);
  si tu workspace usa **enlaces** a los repos (ver paso 5), móntalos en la misma ruta.
- Ya no hacen falta `FLOW_AGENTS_URL` ni `FLOW_AGENTS_TOKEN`: los pone Hilvana.

## 2. Instalar con un clic

**Config ▸ Plugins ▸ Agentes · AgentOffice ▸ Instalar ▸ Instalar con un clic.** Hilvana:

1. descarga `juankanh/agentoffice` (la última versión),
2. crea la red `hilvana-agentoffice` y se conecta a ella,
3. arranca el contenedor `hilvana-agentoffice` con su propio token, el volumen
   `hilvana-agentoffice-data` (datos y sesiones de las CLIs) y tus carpetas,
4. espera a que responda y activa el plugin.

La instalación debe estar **vinculada** a tu cuenta Hilvana (Config ▸ Licencia): sin ello AgentOffice
arranca pero se queda bloqueado.

**Sin acceso a Docker** el panel muestra el `docker run` equivalente, con el token ya preparado,
para que lo ejecutes tú; después pulsa **Comprobar**.

## 3. Iniciar sesión en los agentes (una vez)

```bash
docker exec -it hilvana-agentoffice claude auth login
docker exec -it hilvana-agentoffice codex login    # si también usas Codex
```

La sesión se guarda en el volumen: sobrevive a reinicios y actualizaciones. Para el **Guía** también
puedes poner una clave de API (Anthropic, OpenAI o IA local) en los Ajustes de AgentOffice.

> **Codex por SSH**: `codex login` redirige el navegador a `127.0.0.1:1455` del servidor. Usa
> `codex login --device-auth` si tu versión lo admite, o un túnel `ssh -L 1455:127.0.0.1:1455`.

## 4. Actualizar

En **Config ▸ Plugins** aparece **⬆ Versión nueva X** cuando hay otra imagen publicada:
**Actualizar** recrea el contenedor con ella (los datos y sesiones se quedan en el volumen).

> Dentro del contenedor no hay escritorio: la burbuja de chat, el modo Blender y el control del
> teclado/ratón necesitan la instalación en la máquina (`npm start` del paquete), que sigue
> funcionando como antes.

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

## 6. Alternativa: instalarlo en la máquina (sin contenedor)

Solo si necesitas la burbuja, el modo Blender o el control del escritorio. Descarga el paquete firmado
(con la cuenta vinculada), extráelo en `~/agent-office`, arráncalo con `AO_HOST=0.0.0.0 npm start` y pon en
Hilvana `FLOW_AGENTS_URL=http://host.docker.internal:7420`, `--add-host=host.docker.internal:host-gateway`
y `FLOW_AGENTS_TOKEN` (el de `~/agent-office/data/.token`). Requisitos: Node 20+, git y `claude`/`codex`.

Como servicio de tu usuario, para que arranque con la máquina y se relance tras actualizarse:

```bash
mkdir -p ~/.config/systemd/user
cat > ~/.config/systemd/user/agent-office.service <<'UNIT'
[Unit]
Description=AgentOffice
After=network.target

[Service]
WorkingDirectory=%h/agent-office
Environment=AO_HOST=0.0.0.0
Environment=AO_PORT=7420
# la carpeta bin de tu Node (con nvm: ~/.nvm/versions/node/v22.x.y/bin) debe ir la primera
Environment=PATH=%h/.nvm/versions/node/v22.23.3/bin:/usr/local/bin:/usr/bin:/bin
ExecStart=%h/.nvm/versions/node/v22.23.3/bin/node server/index.js
Restart=on-failure
RestartSec=3

[Install]
WantedBy=default.target
UNIT
systemctl --user daemon-reload && systemctl --user enable --now agent-office
sudo loginctl enable-linger $USER    # que siga corriendo sin una sesión abierta
```

## 7. Se actualiza solo (instalación en la máquina)

Cada 15 min AgentOffice mira su repo de GitHub; si hay versión nueva, **no hay ningún agente trabajando**
y tu copia no tiene cambios propios, hace `git pull`, te avisa (si tienes Telegram) y se reinicia —con el
servicio de systemd o con `npm start`, que usa un lanzador que lo relanza—. Estado:
`curl localhost:7420/api/version`. Para apagarlo: `AO_AUTOUPDATE=off`.

Desde **Hilvana 5.25**, en **Config ▸ Plugins** ves si va por detrás y lo actualizas con **⬆ Actualizar**,
sin esperar a los 15 min. Si AgentOffice no está instalado, ese mismo panel te da la guía con los
comandos. Si lo arrancaste con `node server/index.js` a secas, nadie lo relanzaría: se actualiza en disco
y te avisa de que falta reiniciarlo, en vez de apagarse.

## 8. Avisos al móvil (opcional)

Para enterarte de lo que te necesita (tarea esperando revisión, una pregunta de un agente, sin cuota,
un atasco, un fallo) por **Telegram**, crea `~/agent-office/data/telegram.json`:

```json
{ "enabled": true, "botToken": "123456:ABC…", "chatIds": ["tu_chat_id"] }
```

Prueba: `curl -X POST localhost:7420/api/telegram/test`.

## 9. Revisión de las tareas

Por proyecto (Ajustes ▸ Proyecto) o para todos:

| Política | Qué pasa al terminar una tarea |
|----------|-------------------------------|
| **manual** (por defecto) | espera a que la apruebes o la devuelvas tú |
| **auto-qa** | un revisor IA la revisa: aprueba y fusiona lo que está bien, devuelve solo lo que bloquea; lo que no puede comprobar (levantar servicios, e2e contra un servidor) lo deja anotado; tras 3 devoluciones, te la deja a ti |
| **auto** | se aprueba sola si pasan las verificaciones que declare la tarea |

Antes de revisar, la rama se pone al día con la principal; los choques en ficheros de historial
(`CHANGELOG.md`…) se combinan solos. Lo que toque ficheros sensibles (Dockerfile, `package.json`,
workflows) siempre te espera a ti.

## 10. Qué pueden tocar los agentes

Los agentes trabajan en una **copia aislada de cada repo** (git worktree, una rama por tarea) y no
hacen `push`. Aun así, ejecutan comandos con **tu usuario**: pueden leer otros ficheros de la máquina
y, el QA, los flows de todo el workspace por el MCP de Hilvana. Úsalo en una máquina y un usuario en
los que confíes, y revisa lo que se fusiona.

## 11. Marketplace: compartir roles, skills y agentes

La pestaña **🛒 Marketplace** de AgentOffice sirve para compartir lo que has montado (roles, skills y
agentes) con tu organización de Hilvana o con todo el mundo, y para instalar lo que han publicado
otros.

**Requisitos:** que Hilvana esté **vinculado a tu cuenta cloud** (Config ▸ Licencia) y que
AgentOffice esté conectado con su token (`FLOW_AGENTS_TOKEN`, paso 4). AgentOffice nunca habla con la
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
| «No hay un AgentOffice en http://host.docker.internal:7420 (fetch failed)» | AgentOffice no está arrancado, escucha solo en `localhost` (falta `AO_HOST=0.0.0.0`), o falta `--add-host=host.docker.internal:host-gateway` en el contenedor |
| «AgentOffice pide token» | falta o no coincide `FLOW_AGENTS_TOKEN` con `~/agent-office/data/.token` |
| La oficina abre pero no sale ningún proyecto | el workspace no tiene carpetas de primer nivel, o el contenedor no monta las rutas de los enlaces |
| Una tarea vuelve una y otra vez «Tu rama choca con…» | el agente no consigue resolver el choque: a los 3 intentos se queda esperando que lo mires tú |
| Página en blanco al abrir Hilvana por `http://<IP>` | actualiza Hilvana a 5.21.3 o posterior |
| Los agentes no hacen nada y la oficina dice «sin cuota» | se acabó la ventana de tu suscripción de Claude/Codex: reanudan solos al reiniciarse |
| El Marketplace dice «Esta instalación no está vinculada…» y Hilvana sí lo está | AgentOffice no manda su token a Hilvana (versiones anteriores a la del 9-oct-2026): actualiza AgentOffice y comprueba que `FLOW_AGENTS_TOKEN` coincide con `~/agent-office/data/.token` |
| «Hilvana rechaza a AgentOffice…» en el Marketplace | el `FLOW_AGENTS_TOKEN` del contenedor de Hilvana no es el de `~/agent-office/data/.token` |
| El Guía se queda en «trabajando…» o un botón (Sí de una 🛡) no responde | versiones anteriores a Hilvana 5.26: el navegador tenía agotadas sus 6 conexiones con Hilvana por tener varias ventanas abiertas. Actualiza a 5.26 (las conexiones en vivo van por WebSocket); mientras, cierra otras ventanas de Hilvana y pulsa ■ Parar. El Guía avisa a los 10 s si un mensaje no sale |
| El navegador del agente abre una ventana aparte | AgentOffice anterior a oct-2026: ahora va oculto y se ve en 🌐 Navegador (`AO_BROWSER_HEADLESS=0` para la ventana) |
| Un rol publicado en Público no aparece | está **⏳ pendiente de moderación**: hasta que se aprueba solo lo ve tu organización |

Más detalle técnico en el README de AgentOffice (repositorio privado: solo con acceso anticipado, ver el
aviso del paso 2).
