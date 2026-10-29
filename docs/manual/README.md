# 📘 Hilvana — Manual de uso

Manual completo de **Hilvana**: la herramienta visual para componer, ejecutar y **documentar** flujos de peticiones HTTP, SQL y navegación web. Todas las capturas son de la aplicación real.

> [!NOTE]
> Manual de **Hilvana v6.3.2** (imagen `juankanh/hilvana:6.3.2`). La 6.3.2 es de documentación (HI-426): sin cambios de comportamiento en la app, deja al día lo de AgentOffice 0.3.13–0.3.17 ([15 AgentOffice](15-agentoffice.md)). La 6.3.1 deja que AgentOffice en Docker use el MCP de Hilvana y que el Taller del Guía trabaje con tu sesión de Claude Code ([15 AgentOffice](15-agentoffice.md)). La 6.3.0 deja ver un flow en PDF desde el móvil. La 6.2.1 pide vincular la cuenta antes de instalar AgentOffice en Docker (y luego sigue solo). La 6.2.0 instala AgentOffice en tu PC (nativo, por defecto) o en un servidor con Docker ([15 AgentOffice](15-agentoffice.md)). La 6.1.0 instala AgentOffice con un clic como contenedor Docker ([15 AgentOffice](15-agentoffice.md)). La 6.0.0 es de mantenimiento (marca Hilvana y preparativos internos; sin cambios de comportamiento en la app). La 5.39.0 amplió el manual pantalla a pantalla. La 5.38.3 añadió a AgentOffice el recuerdo de cada agente, la sesión viva entre tareas y «que siga con…» ([15 AgentOffice](15-agentoffice.md)). Manual en PDF: [hilvana-manual.pdf](hilvana-manual.pdf) y [agentoffice-manual.pdf](agentoffice-manual.pdf).

## Índice

1. [01 La pantalla principal](01-pantalla-principal.md) — toolbar, barra lateral, pestañas, canvas, minimapa, split 🆕
2. [02 El nodo Request](02-nodo-request.md) — las cajitas curl: ejecutar, respuesta, extracciones
3. [03 Conexiones y variables](03-conexiones-y-variables.md) — encadenar nodos y mover datos entre ellos
4. [04 Notas, Mermaid y Capturas](04-notas-mermaid-capturas.md) — documentación dentro del canvas 🆕
5. [05 El nodo SQL](05-nodo-sql.md) — consultas Postgres/MySQL/Oracle en el flujo
6. [06 El nodo Web y el modo Live](06-nodo-web-live.md) — navegar una web embebida y capturar pasos 🆕
7. [07 Documentar una web — flow-explore](07-flow-explore.md) — el CLI que genera documentación sola 🆕
8. [08 Paneles y utilidades](08-paneles.md) — **proyecto (flows/ + Ctrl+S)**, **pizarra**, **correo de prueba (SMTP)**, guía de nodos, historial, consola, batch, GitHub, export 🆕
9. [09 El CLI flow-run](09-cli-flow-run.md) — ejecutar flows desde terminal / CI
10. [10 El MCP embebido](10-mcp.md) — que una IA construya flows en tu canvas
11. [11 Docker y despliegue](11-docker.md)
12. [12 La vista móvil](12-movil.md) — ejecutar flows y leerlos desde el teléfono 🆕
13. [13 El asistente de IA](13-asistente-ia.md) — el chat que construye y arregla flows en tu canvas con tu clave de Claude u OpenAI
14. [14 Reportar un problema](14-reportar-problemas.md) — errores, ideas y preguntas al equipo desde la app, con captura y datos técnicos 🆕
15. [15 AgentOffice](15-agentoffice.md) — la oficina de agentes pantalla a pantalla: Mesa de trabajo (seguimiento en vivo), tareas, equipo, ajustes, revisión, burbuja del agente, modos de desarrollo y móvil 🆕

## Arrancar la aplicación

**En desarrollo** (con todas las funciones, incluidas las 🆕):

```bash
git clone https://github.com/jks90/hilvana && cd hilvana && npm install
npm run dev        # web en http://localhost:5173 + server en :3001
```

**Con Docker** (la imagen trae Chromium: captura y Live incluidos; solo `flow-explore --manual/--headful` necesita un display):

```bash
docker run -d --add-host=host.docker.internal:host-gateway \
  -p 9998:3001 --name hilvana juankanh/hilvana:6.3.2
# web en http://localhost:9998
```

## La idea en 30 segundos

- Cada **cajita** del canvas es un paso: una petición HTTP (curl), una consulta SQL, una nota, una captura de pantalla o una web.
- Las **flechas** definen el orden de ejecución; las **variables** `{{asi}}` mueven datos de una cajita a la siguiente.
- Un flujo se guarda como un `.flow.json` que se puede versionar, compartir, y ejecutar **sin abrir el navegador** con el CLI `flow-run`. 🆕 Desde la 4.25 la carpeta `flows/` del servidor es el **proyecto**: la web sabe qué fichero es cada pestaña y **Ctrl+S** lo guarda ahí.
- Además de probar APIs, sirve para **documentarlas**: screenshots de cada pantalla de una web con sus llamadas HTTP debajo, generados navegando (modo Live) o con el explorador automático (`flow-explore`).
- 🆕 Y para **explicarlas encima del propio lienzo**: la pizarra deja rodear, anotar y señalar las cajas, y la guía de nodos te lleva a cualquier caja de un clic.
