# ⌨️ 09 · El CLI flow-run

Los mismos `.flow.json` del canvas, ejecutados **desde terminal**: para CI, cron del sistema, o baterías de regresión. Sin navegador.

```bash
cd hilvana
npm run flow:run -- flows/api-store.flow.json    # un flow
npm run flow:batch -- flows                      # todos los de una carpeta (recursivo)

# o directamente / instalado global con npm link:
node cli/run-flow.js --flow flows/mi.flow.json --var apiBase=http://localhost:8080
flow-run /ruta/a/flows
```

- Ejecuta los nodos en el **mismo orden topológico** que la web, con extracciones e interpolación — paridad total.
- **Exit code 1 si algo falla** → válido como test de CI.
- Ejecuta también los **sqlNodes** (perfiles: `--sql-connections <file>` o el `sql-connections.json` más cercano al flow).

## Flags

| Flag | Qué hace |
|------|----------|
| `--flow <file>` / `--dir <dir>` | Un flow o una carpeta entera (ordenada por nombre) |
| `--var clave=valor` | Añade/sobrescribe variables (repetible) |
| `--timeout <ms>` | Timeout por petición (defecto 180000) |
| `--continue-on-error` | Sigue con el resto de nodos tras un error |
| `--skip-sql-nodes` | Ignora los sqlNodes (¡un batch toca BBDD reales!) |
| `--sql-connections <file>` | Perfiles SQL |
| `--out <file-o-dir>` | Log JSON de la ejecución |
| `--report-root <dir>` / `--no-report` | Carpeta de informes (defecto `resumen/`) o desactivarlos |
| `--env <nombre>` 🆕 5.3 | Entorno con nombre: aplica `environments[nombre]` sobre las variables (sin la flag, vale el entorno activo guardado en el flow) |
| `--data <fichero>` 🆕 5.3 | Data-driven: CSV (primera fila = nombres de variable) o JSON array — el flow corre una vez por fila con sus columnas como variables; columna `name` etiqueta el caso. Requiere `--flow` |
| `--no-localhost-rewrite` | No reescribir localhost→host.docker.internal (dentro de Docker) |

> **`-F` con `;type=`** (5.27): en una petición multipart, un campo de texto con tipo (`-F 'meta={"a":1};type=application/json'`) envía ya la cabecera `Content-Type` de esa parte, como hace curl (lo necesitan p. ej. los `@RequestPart` JSON de Spring). La web sigue sin enviar multipart: esos flows ejecútalos con el CLI.

## Informes

Cada ejecución deja en `resumen/<fecha-hora>/` un **markdown** por flow con el detalle petición/respuesta de cada nodo, más un índice. Los informes son la referencia de «qué pasó» sin tener la web abierta.

> [!TIP]
> **Los flows como tests**
> El patrón que usamos en el Core y la tienda: una batería de flows por API en `flows/`, y `flow-run --dir flows` en CI. Si un endpoint rompe el contrato, el flow falla y el informe te dice exactamente dónde.

## Asserts, data-driven y CI 🆕 (5.3)

- Los **asserts por nodo** (definidos en la app — [02](02-nodo-request.md) del manual) se evalúan también en el CLI: un assert KO marca el nodo como fallido, se imprime bajo la caja (`✗ assert: $.rates.USD exists — real: (sin valor)`) y el proceso **sale con código ≠ 0** — listo para el pipeline.
- **Data-driven**: `flow-run --flow api.flow.json --data casos.csv` ejecuta el flow una vez por fila; el resumen agrega los casos (`Summary: 1/3 flows passed`) y cada fallo sale con su nombre de caso.
- **En CI**: el runner va dentro de la imagen — `docker run --rm -v "$PWD/flows:/app/flows" juankanh/hilvana node cli/run-flow.js --dir flows` (función **Pro**: pasa tu licencia con `-e FLOW_LICENSE=…` desde los secrets del CI). El paquete npm del runner (`flow-run` + `flow-bridge`, el puente cloud → tu máquina) **aún no está publicado**.

## Todos los flags 🆕 (5.38.1)
Entrada: `--flow <f>` (un flow) · `--dir <d>` (lote, recursivo) · `--help`.
Datos y variables: `--var k=v` (sobrescribe variables) · `--env <entorno>` (entorno de `Flow.environments`) · `--data <fichero>` (data-driven: una pasada por fila).
Ejecución: `--timeout` (por defecto 180 s) · `--continue-on-error` · `--no-localhost-rewrite` (en Docker `localhost` → `host.docker.internal`) · `--skip-sql-nodes` · `--skip-info-scripts`.
SQL: `--sql-connections <fichero>` (perfiles; por defecto `flows/sql-connections.json`).
Salida: `--out` · `--report-root` · `--no-report` (informes en `resumen/`). Exit 1 si algo falla.
