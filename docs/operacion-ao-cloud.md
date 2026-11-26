# Operación del AgentOffice en el cloud (guía interna · HI-552)

Documento para el admin del cloud. Qué variables gobiernan los AgentOffice por org, cómo guardar y rotar `AO_SECRETS_KEY` y cómo actualizar la imagen. Los nombres, tablas y rutas están verificados contra `hilvana-accounts` (`lib/ao-orchestrator.js`, `lib/secrets.js`, `lib/ao-credentials.js`, `lib/org-agentoffice.js`, `scripts/orchestrator.mjs`). Qué hace cada pieza (contenedor, red, cupo, ciclo de vida) está en `flows/hilvana/README-HI-498.md` del repo hilvana y en el README de accounts («Orquestador de AgentOffice por org» y «Cola de operaciones»); aquí no se repite.

> **Versión 6.11.0 (HI-596):** esta guía aplica a la entrega completa del AgentOffice en la nube con AgentOffice 0.3.32 (Hilvana 6.11.0 no cambia la app); lo que ve el cliente está en `docs/manual/16-agentoffice-nube.md`.

> Esta guía solo describe; no ejecuta nada. Los comandos son para el VPS (`ssh` del admin) y para nadie más. Nunca pegues valores reales de claves en tickets, chats ni logs.

## 1. Variables

Se definen en el `.env` de accounts del VPS (`/opt/hilvana/hilvana-accounts/.env`; el compose canónico es `/opt/hilvana/docker-compose.yml`). Las lee el servidor `hilvana-accounts` (que arma los argumentos de `docker create`) y el orquestador del host ejecuta lo que el servidor encola.

| Variable | Defecto | Efecto |
| --- | --- | --- |
| `ORG_AO_IMAGE` | — (sin ella: 500 «ORG_AO_IMAGE sin definir» al crear) | Imagen con la que se **crea** el contenedor `hilvana-ao-<slug>`, p. ej. `juankanh/agentoffice:<versión>`. Solo se usa en `create`: los contenedores ya creados no cambian al cambiarla (ver §3). |
| `AO_CLOUD_MEM` | `2g` | `--memory` de cada contenedor AO (`2g`, `512m`…). |
| `AO_CLOUD_CPUS` | `1` | `--cpus` de cada contenedor AO. |
| `AO_CLOUD_MAX_ACTIVE` | `2` | Cupo global de AO **despiertos** (estado confirmado `running`). Superado → 409 «Cupo de AgentOffice agotado…». |
| `AO_CLOUD_MAX_AGENTS` | `3` | Se pasa al AO como `AO_CLOUD_MAX_AGENTS`: agentes simultáneos por AO. |
| `AO_SECRETS_KEY` | — (obligatoria) | Clave AES-256-GCM (32 bytes: 64 hex o base64). Cifra en la BD el `AO_TOKEN` de cada org y sus credenciales de IA y de repos. Sin ella, o inválida, esas rutas fallan (500) y no guardan nada. |

Los cambios de `AO_CLOUD_MEM`, `AO_CLOUD_CPUS` y `AO_CLOUD_MAX_AGENTS` solo afectan a los contenedores **creados después** (van en `docker create`); para aplicarlos a uno existente hay que recrearlo (§3). `AO_CLOUD_MAX_ACTIVE` se aplica en cuanto accounts reinicia.

Tras editar el `.env`, reinicia accounts (el compose lo relee al recrear el servicio):

```bash
cd /opt/hilvana && docker compose -f /opt/hilvana/docker-compose.yml up -d hilvana-accounts
```

### Generar y guardar `AO_SECRETS_KEY`

```bash
openssl rand -hex 32          # 64 caracteres hex; apúntala solo en el .env y en la copia de abajo
```

1. Añade `AO_SECRETS_KEY=<la clave>` al `.env` de accounts del VPS (permisos 600, sin comillas) y reinicia accounts como arriba.
2. **Copia de seguridad de la clave, junto a `copias.key`:** en el VPS `/opt/hilvana/ao-secrets.key` (600) y fuera del VPS, en katana, `~/.config/hilvana/ao-secrets.key`, donde ya está la otra: `copias.key` (ver `scripts/prod-seguridad/README.md` del repo hilvana). Las copias diarias cifradas incluyen el `.env`, pero sin la clave las filas cifradas de la BD **no se pueden recuperar**: una clave perdida equivale a que todas las orgs tengan que volver a meter sus credenciales y a perder los tokens de sus AO.
3. Comprueba que el `.env` la tiene y que accounts la acepta sin imprimirla: `docker exec hilvana-accounts node -e "import('./lib/secrets.js').then(m=>console.log(m.decrypt(m.encrypt('ok'))))"` debe escribir `ok` (si no, sale «AO_SECRETS_KEY ausente o inválida»).

## 2. Rotar `AO_SECRETS_KEY`

**No existe ningún script de rotación en `hilvana-accounts`** (verificado: ni en `scripts/` ni en `lib/`). Hay dos caminos; el primero es el recomendado.

Qué cifra la clave (formato `v1:<iv>:<tag>:<texto>` en base64): `org_agentoffice.token_enc` (el `AO_TOKEN` de cada org), `org_ao_ai.token_enc` y `org_ao_repos.token_enc`. Rotar **no** exige recrear contenedores: el `AO_TOKEN` en claro no cambia (los contenedores ya lo llevan en su entorno) y los ficheros de `secrets/` del volumen están en claro, no con esta clave.

### Camino A · descifrar con la antigua y recifrar con la nueva (sin molestar a los clientes)

1. Haz una copia de la BD antes (`/opt/hilvana/seguridad/copia.sh`, o `docker exec mysql mysqldump …` como ya uses) y comprueba que existe.
2. Genera la nueva: `NEW=$(openssl rand -hex 32)` y guárdala ya (§1, punto 2) **sin** cambiar aún el `.env`.
3. Guarda este script como `/tmp/rotar-ao-key.mjs` en el VPS. Usa las funciones reales de `lib/secrets.js` (que leen `AO_SECRETS_KEY` en cada llamada) y las tablas reales. Sin `APLICAR=1` solo comprueba; con él, reescribe todo en **una transacción** (o no cambia nada).

```js
// HI-552 · rotación de AO_SECRETS_KEY: OLD_KEY descifra, NEW_KEY recifra. Se ejecuta dentro de hilvana-accounts (cwd = raíz del app).
const { pool } = await import('./lib/db.js');
const { encrypt, decrypt } = await import('./lib/secrets.js');
const { OLD_KEY, NEW_KEY, APLICAR } = process.env;
if (!OLD_KEY || !NEW_KEY || OLD_KEY === NEW_KEY) throw new Error('define OLD_KEY y NEW_KEY distintas');
const rot = (enc) => { process.env.AO_SECRETS_KEY = OLD_KEY; const p = decrypt(enc); process.env.AO_SECRETS_KEY = NEW_KEY; return encrypt(p); };
const TABLAS = [['org_agentoffice', 'org_id'], ['org_ao_ai', 'org_id, provider'], ['org_ao_repos', 'id']];
const c = await pool.getConnection();
let n = 0;
try {
  await c.beginTransaction();
  for (const [t, pk] of TABLAS) {
    const [rows] = await c.query(`SELECT ${pk}, token_enc FROM ${t} WHERE token_enc IS NOT NULL`);
    for (const r of rows) {
      const nuevo = rot(r.token_enc); // si falla aquí, la clave antigua no es la correcta: se aborta y no se toca nada
      const where = pk.split(', ').map((k) => `${k}=?`).join(' AND ');
      await c.query(`UPDATE ${t} SET token_enc=? WHERE ${where}`, [nuevo, ...pk.split(', ').map((k) => r[k])]);
      n++;
    }
  }
  if (APLICAR === '1') { await c.commit(); console.log(`recifradas ${n} filas`); } else { await c.rollback(); console.log(`simulación OK: ${n} filas se pueden recifrar (nada cambiado)`); }
} catch (e) { await c.rollback(); console.error('ABORTADO, nada cambiado:', e.message); process.exitCode = 1; } finally { c.release(); await pool.end(); }
```

4. Simulación (no cambia nada). La clave actual está en el `.env`; no la escribas en el historial de la shell:

```bash
OLD=$(grep -E '^AO_SECRETS_KEY=' /opt/hilvana/hilvana-accounts/.env | cut -d= -f2-)
docker exec -i -w /app -e OLD_KEY="$OLD" -e NEW_KEY="$NEW" hilvana-accounts node --input-type=module < /tmp/rotar-ao-key.mjs
```

5. Si dice «simulación OK», aplica y, **inmediatamente después**, cambia el `.env` y reinicia accounts (entre ambos pasos accounts no puede descifrar: hazlo seguido, mejor en horario de poco uso):

```bash
docker exec -i -w /app -e APLICAR=1 -e OLD_KEY="$OLD" -e NEW_KEY="$NEW" hilvana-accounts node --input-type=module < /tmp/rotar-ao-key.mjs
sed -i "s/^AO_SECRETS_KEY=.*/AO_SECRETS_KEY=$NEW/" /opt/hilvana/hilvana-accounts/.env
cd /opt/hilvana && docker compose -f /opt/hilvana/docker-compose.yml up -d hilvana-accounts
```

6. Verifica: el comando del §1 punto 3 devuelve `ok`; el admin `GET /api/admin/agentoffice/<slug>/token` (desde el VPS, `127.0.0.1`) devuelve un token; un AO de prueba arranca. Actualiza las dos copias de la clave (`/opt/hilvana/ao-secrets.key` y la de katana) y borra `/tmp/rotar-ao-key.mjs`. Conserva la clave antigua hasta que la copia cifrada de esa noche incluya el `.env` nuevo (las copias anteriores solo se abren con la antigua).

### Camino B · pedir a los clientes que vuelvan a meter sus credenciales

Solo si la clave antigua **se ha perdido** (Camino A imposible) o está comprometida y se quiere invalidar lo guardado:

1. Cambia `AO_SECRETS_KEY` en el `.env` y reinicia accounts (§1).
2. Los `AO_TOKEN` guardados dejan de descifrarse. Para cada org con AO: `docker rm -f hilvana-ao-<slug>` en el host y vacía su token (`UPDATE org_agentoffice SET token_enc=NULL WHERE slug='<slug>'` en la BD). Accounts **no** lo regenera solo (`ensureToken` solo corre al levantar una fila sin contenedor), y sin token `GET …/token` da 404. Genera uno nuevo a mano (debe imprimir `true`):
   ```bash
   docker exec -w /app hilvana-accounts node --input-type=module -e "const {pool}=await import('./lib/db.js');const a=await import('./lib/org-agentoffice.js');const [r]=await pool.query('SELECT org_id FROM org_agentoffice WHERE slug=?',['<slug>']);console.log(await a.ensureToken(r[0].org_id));await pool.end()"
   ```
   Después recrea el contenedor con los pasos del §3 (que piden el token a `GET …/token`). El `data` del volumen se conserva. (HI-552)
3. Las credenciales de IA y de repos (`org_ao_ai`, `org_ao_repos`) no se pueden recuperar: borra las filas (`DELETE FROM org_ao_ai; DELETE FROM org_ao_repos;`) y avisa a los clientes para que las vuelvan a introducir en su cuenta (`PUT /api/orgs/:slug/ao/ai`, `POST /api/orgs/:slug/ao/repos`); al guardarlas se vuelven a entregar al volumen.

## 3. Actualizar la imagen del AgentOffice

`ORG_AO_IMAGE` solo se usa al crear el contenedor. Cambiarla **no** actualiza los que ya existen; hay que recrearlos. El volumen (`<ORG_DATA_HOST_DIR>/ao/<slug>`, montado en `/data`) es un bind mount: borrar el contenedor no toca los datos.

> **No uses `POST /api/admin/agentoffice/<slug>/delete`**: es `purge` y borra `/data`. Tampoco hay ruta de API para «recrear sin purgar»; se hace en el host con `docker`, replicando `aoCreateArgs`.

1. Comprueba que la imagen nueva existe en el VPS: `docker pull <imagen:versión>` (y anota la versión actual: `docker ps --filter name=hilvana-ao- --format '{{.Names}}\t{{.Image}}'`).
2. En el `.env` de accounts cambia `ORG_AO_IMAGE=<imagen:versión>` y reinicia accounts (§1). Los AO **nuevos** ya salen con ella.
3. Para cada org con AO ya creado (lista: `docker ps -a --filter name=hilvana-ao- --format '{{.Names}}\t{{.Status}}'`), recrea conservando `/data`. Haz una a una, en horario de poco uso: se interrumpe el AO de esa org (tareas `doing` incluidas) unos segundos. Variables: `SLUG` de la org, `ADM` el `ADMIN_TOKEN` (del mismo `.env`), `IMG` la imagen nueva, y `MEM`, `CPUS`, `AGENTS`, `HOSTDATA` (= `ORG_DATA_HOST_DIR` o `ORG_DATA_DIR`) con los valores del `.env` (defectos 2g, 1, 3):

```bash
SLUG=demo-org; IMG=juankanh/agentoffice:<versión>; MEM=2g; CPUS=1; AGENTS=3; HOSTDATA=/opt/hilvana/org-data
ADM=$(grep -E '^ADMIN_TOKEN=' /opt/hilvana/hilvana-accounts/.env | cut -d= -f2-)
WAS_UP=$(docker ps -q --filter "name=^hilvana-ao-$SLUG$")
export AO_TOKEN=$(curl -s -H "authorization: Bearer $ADM" http://127.0.0.1:7100/api/admin/agentoffice/$SLUG/token | node -pe 'JSON.parse(require("fs").readFileSync(0)).token')
docker stop -t 10 hilvana-ao-$SLUG && docker rm hilvana-ao-$SLUG          # sin -v: el volumen es un bind mount y se conserva
docker create --name hilvana-ao-$SLUG --network hilvana-org-$SLUG-net -v $HOSTDATA/ao/$SLUG:/data \
  --memory $MEM --cpus $CPUS --pids-limit 512 --cap-drop ALL --security-opt no-new-privileges \
  -e AO_TOKEN -e AO_MODE=cloud -e AO_CLOUD_MAX_AGENTS=$AGENTS -e AO_FLOW_URL=http://hilvana-org-$SLUG:3001 $IMG
unset AO_TOKEN
[ -n "$WAS_UP" ] && docker start hilvana-ao-$SLUG                         # si estaba dormido/parado se deja como estaba
```

   Si el `.env` define `AO_CLOUD_USER` (uid[:gid]), añade `--user <uid[:gid]>` al `docker create`. El `AO_TOKEN` va por entorno (`-e AO_TOKEN`), nunca en los argumentos. Los flags son los de `aoCreateArgs` (`lib/ao-orchestrator.js`); si ese fichero cambia en una versión futura, revisa que sigan iguales.
4. Comprueba cada org:

```bash
docker ps --filter name=hilvana-ao- --format '{{.Names}}\t{{.Image}}\t{{.Status}}'        # imagen nueva y Up
docker exec hilvana-ao-$SLUG node -e "fetch('http://127.0.0.1:7420/api/version').then(async r=>console.log(r.status,await r.text()))"   # 200 y la versión nueva
ls $HOSTDATA/ao/$SLUG                                                                      # los datos siguen ahí
curl -s -H "authorization: Bearer $ADM" http://127.0.0.1:7100/api/admin/agentoffice | head -c 600   # estado y running/max
```

5. Si algo falla: vuelve `ORG_AO_IMAGE` a la versión anterior, reinicia accounts y repite el paso 3 con la imagen anterior (el volumen no se ha tocado; si la versión nueva migró los datos, restaura la copia de `org-data` de esa noche).

## 4. Memoria en el VPS de 8 GB

Cada AO despierto puede llegar a `AO_CLOUD_MEM` (2 GB por defecto) y `AO_CLOUD_MAX_ACTIVE` limita cuántos están despiertos a la vez. Con los defectos, el techo de los AO es 2 × 2 = 4 GB, y el resto del VPS (MariaDB, accounts, Caddy, los Hilvana de las orgs y el sistema) tiene que caber en lo que quede. Reglas:

- **No subas `AO_CLOUD_MAX_ACTIVE` ni `AO_CLOUD_MEM` sin mirar la memoria libre** con todo el cloud en uso. Antes de pasar de 2 a 3 despiertos con 2 GB, comprueba que quedan ≥ 2 GB de margen.
- `AO_CLOUD_MEM` es un tope duro del contenedor: al pasarse, Docker mata el proceso (OOM) y el AO se reinicia sin la tarea en curso.
- **Vigilancia:** `estado.py` (cron cada minuto) escribe `/opt/hilvana/estado/estado.json` con CPU, memoria, disco y contenedores principales (ver `scripts/prod-seguridad/README.md` en hilvana); el vigía de Nilo en serverman lo mira cada 15 min y avisa por Telegram. Comprobaciones a mano:

```bash
free -h                                                                             # memoria total / disponible
docker stats --no-stream --format '{{.Name}}\t{{.MemUsage}}\t{{.MemPerc}}' | sort   # uso real por contenedor (los hilvana-ao-* contra su límite)
docker ps -a --filter name=hilvana-ao- --format '{{.Names}}\t{{.Status}}'            # cuántos AO están Up (≤ AO_CLOUD_MAX_ACTIVE)
dmesg -T | grep -i 'out of memory' | tail                                            # muertes por OOM (necesita sudo)
```

- Si la memoria disponible baja de ~1 GB de forma sostenida: duerme o para el AO de menos uso (`POST /api/admin/agentoffice/<slug>/stop`, **no** `delete`), baja `AO_CLOUD_MAX_ACTIVE` a 1 o `AO_CLOUD_MEM` a `1g` (este último solo afecta a los AO que se creen después), y reinicia accounts.

## 5. Ejecución en el host y recreación del Hilvana (6.11.0 · HI-596)

El servidor `hilvana-accounts` **no habla con Docker**: cada operación (crear, despertar, parar, borrar) se encola en la tabla `ao_ops` y la ejecuta el **orquestador del host** (`scripts/orchestrator.mjs`, servicio aparte; sondea cada `ORCHESTRATE_EVERY_MS`, 15 s por defecto, contra `FLOW_ACCOUNTS_URL`).

- **Token del AO:** el orquestador lo pide con `GET /api/admin/agentoffice/<slug>/token`, que solo responde a llamadas desde el propio host (`127.0.0.1` o `AO_HOST_CALLER_IPS`; 403 en otro caso) y no se cachea. Se entrega al contenedor por entorno.
- **Confirmación:** al terminar, el orquestador llama a `POST /api/admin/agentoffice/<slug>/result` con `{status, message}`. El estado de la página de cuenta solo cambia con esta confirmación; repetirla no duplica nada.
- **Recreación del Hilvana de la org:** al confirmarse `running` o `stopped`, accounts recrea el Hilvana de la organización conservando sus flows: con `AGENTOFFICE_MANAGED=cloud`, `AGENTOFFICE_URL` y `AGENTOFFICE_TOKEN` si el AO queda en marcha, sin ellas si se para. El cliente lo nota como una recarga de unos segundos.
- **Ciclo de vida:** activo salvo `AO_LIFECYCLE=0`; solo encola operaciones (duerme sin tareas `doing`, para al dejar Business, purga a los 30 días).
- **Si un AO no cambia de estado:** mira primero que el servicio del orquestador esté en marcha (`09-health` lo vigila como `hilvana-orchestrator`) y que la cola `ao_ops` no tenga operaciones atascadas; después los registros del orquestador. Los contenedores salen como `hilvana-ao-<slug>`.

## 6. Puerta de entrega de AgentOffice 0.3.32

La imagen del AO que fija `ORG_AO_IMAGE` debe ser la 0.3.32 para que los clientes tengan la puerta de entrega (ver «Cuándo una tarea se considera hecha» en el cap. 16): una tarea de código sin cambios ni resumen vuelve una vez a «Por hacer» y la segunda llega a Revisión como «No está hecha»; el agente cierra con `ENTREGA: COMPLETA|PARCIAL|NO HECHA` y las dos últimas nunca se aprueban solas. Los contenedores ya creados no cambian al cambiar `ORG_AO_IMAGE`: sigue la recreación de la §3.
