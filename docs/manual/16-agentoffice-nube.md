# 16 · AgentOffice en la nube (Business)

> Revisado para Hilvana **6.9.0** · Texto de la página de tu cuenta ▸ **🏢 AgentOffice en la nube**. Sin capturas: la página de la cuenta no forma parte de la aplicación local.

<a id="que-es"></a>
## Qué es y para quién

Con el plan **Business**, tu organización puede tener un **AgentOffice alojado por Hilvana cloud**: no instalas nada, no necesitas Docker ni dejar un PC encendido. El equipo de agentes (Claude Code y Codex) trabaja en tus repos igual que el [AgentOffice local](15-agentoffice.md), pero corre en la nube y **duerme solo** cuando no hay trabajo.

- Solo para **organizaciones Business** (incluida una cortesía Business vigente y sin caducar).
- Solo lo gestionan el **propietario (owner)** y los **administradores (admin)** de la organización; el resto de miembros no ve la sección.

<a id="requisitos"></a>
## Requisitos

1. Organización con plan **Business** vigente. Con otro plan, la sección aparece con 🔒, el aviso «AgentOffice en la nube solo está disponible en el plan Business» y el botón **Mejorar a Business**, que te lleva a Suscripción.
2. Entrar en tu cuenta de Hilvana cloud como owner o admin de la organización.
3. Una **cuenta de IA propia**: token de Claude, clave de Anthropic o clave de OpenAI (ver [la IA del AgentOffice](#token-ia)). Los agentes gastan tu cuota, no la de Hilvana.
4. Opcional pero habitual: un **token de repo** por cada repositorio https en el que quieras que trabajen (ver [repos](#token-repo)).

<a id="activarlo"></a>
## Activarlo desde la cuenta

1. En la página de tu cuenta abre, en el menú lateral, **🏢 AgentOffice en la nube**.
2. Activa el interruptor **AgentOffice activado**. Para pararlo, desactívalo.
3. Configura la IA y los repos (secciones siguientes).

El estado se refresca cada 10 s y muestra:

| Estado | Significado |
|---|---|
| 🟢 despierto | Está en marcha. |
| 🌙 dormido | Sin actividad y sin tareas en curso: se duerme solo y despierta cuando hace falta. |
| ⚫ parado | Desactivado, o la organización dejó de ser Business. |

Además verás **RAM**, **Última actividad** y **Mesas en curso**. El estado solo cambia cuando el servidor confirma la operación, así que tras activar o desactivar puede tardar unos segundos.

<a id="token-ia"></a>
## La IA del AgentOffice (🔑)

En **🔑 IA del AgentOffice** elige el tipo, pega el valor en **Pega el token o la clave** y pulsa **Guardar**:

| Opción | Variable que se configura | Cómo conseguirlo |
|---|---|---|
| **Token de Claude (setup-token)** | `CLAUDE_CODE_OAUTH_TOKEN` | En una terminal con Claude Code instalado y tu sesión iniciada, ejecuta `claude setup-token`, completa el acceso en el navegador y copia el token que imprime. Usa tu suscripción de Claude. |
| **Clave de API de Anthropic** | `ANTHROPIC_API_KEY` | Créala en la consola de Anthropic (API keys). Se factura por uso. |
| **Clave de API de OpenAI** | `OPENAI_API_KEY` | Créala en la plataforma de OpenAI (API keys). Es la que usa **Codex**. |

El valor se guarda **cifrado** y se entrega una sola vez al AgentOffice; la página no vuelve a mostrarlo, solo «configurado ✓». Para cambiarlo, vuelve a pegar uno nuevo y **Guardar**.

<a id="token-repo"></a>
## Repos y token de repo con permisos mínimos (📦)

En **📦 Repos** escribe la **URL https** del repositorio y su **Token del repo**, y pulsa **Añadir**. **Quitar** lo elimina. Si dos repos del mismo servidor llevan tokens distintos, se usa el más reciente (la página te avisa). El token se guarda cifrado y no se vuelve a mostrar.

Crea siempre un token **solo para esto**, con el mínimo imprescindible y con caducidad. La cuenta no valida el token: si es incorrecto lo notarás cuando un agente intente clonar o subir una rama.

**GitHub (fine-grained token)**
1. GitHub ▸ Settings ▸ Developer settings ▸ Personal access tokens ▸ **Fine-grained tokens** ▸ Generate new token.
2. **Repository access ▸ Only select repositories**: marca solo los repos necesarios.
3. **Permissions ▸ Repository ▸ Contents: Read and write** (Metadata: read lo añade GitHub solo). Nada más, salvo que quieras que abran pull requests (entonces, además, Pull requests: Read and write).
4. Fija una caducidad, genera y copia el token.

**GitLab**
1. En el proyecto (o grupo) ▸ Settings ▸ Access tokens ▸ Add new token.
2. Rol **Developer** y permisos `read_repository` y `write_repository`.
3. Fija caducidad y copia el token.

<a id="en-hilvana"></a>
## Qué verás en Hilvana

El Hilvana de tu organización (el que crea el cloud) muestra en el panel de agentes **☁ Gestionado por Hilvana cloud**: no hay botones de instalar ni actualizar, porque se gestiona desde la cuenta. Si ese Hilvana no tiene plan Business, aparece «Los agentes gestionados son una función Business» con el botón **Ir a mi cuenta**.

<a id="errores"></a>
## Errores típicos

- **La organización no es Business vigente**: no se puede activar; mejora el plan o renueva.
- **Cupo de AgentOffice despiertos alcanzado**: el servicio tiene un límite global de oficinas despiertas a la vez; reintenta más tarde.
- **Red de la organización inexistente**: el Hilvana de la org aún no está provisionado; el mensaje se muestra tal cual y se resuelve al terminar su aprovisionamiento.
- **No ves la sección**: no eres owner/admin de la organización.

<a id="desactivar"></a>
## Desactivar y dejar Business

Al desactivar, al dejar el plan Business o al caducar, el AgentOffice se **para** y sus datos se **conservan 30 días**; si no vuelves a activarlo en ese plazo, se borran.

<a id="limites"></a>
## Límites

Cada organización tiene su propio AgentOffice con estos valores por defecto del servicio:

| Límite | Valor por defecto | Qué significa |
|---|---|---|
| Memoria (RAM) | **2 GB** | Tope del contenedor; es la cifra de **RAM** que ves en la página cuando está despierto. |
| CPUs | **1** | Un procesador por AgentOffice. |
| Mesas | **3** | Máximo de mesas de trabajo (tareas de agentes) a la vez en tu organización. |
| Cupo de despiertos | **2** | Máximo de AgentOffice **despiertos a la vez en todo el servicio**, no por organización. |

- **Cupo superado.** Si ya hay el máximo de AgentOffice despiertos, al activar o despertar el tuyo la página muestra un aviso (error 409): «Cupo de AgentOffice agotado: ya hay N despiertos (máximo M)…». No es un fallo de tu cuenta: espera a que otro duerma y reintenta.
- **Dormir solo.** Tras **30 minutos** sin actividad, el AgentOffice se duerme; vuelve a despertar cuando lo usas. **No se duerme mientras tenga una tarea en curso** (si no se puede comprobar, tampoco se duerme y se reintenta).
- **Al dejar Business** (o caducar el plan) se **para** y empieza la cuenta atrás de borrado ([más abajo](#borrado)).

Los valores pueden ser distintos en una instalación propia del servicio; los de arriba son los de serie.

<a id="datos"></a>
## Qué datos se guardan

- **Credenciales** (token o clave de IA y tokens de repo): **cifradas con AES-256-GCM**. La API de la cuenta nunca las devuelve (solo «configurado ✓») y no se escriben en los registros.
- **Volumen de datos de tu AgentOffice**: una carpeta propia de tu organización con tus **clones de repos, las copias de trabajo (worktrees) de cada tarea**, tareas, equipo y ajustes. Persiste aunque el AgentOffice duerma.

<a id="borrado"></a>
## Cómo se borran

- **Al dejar Business o caducar**: el AgentOffice se para y los datos se conservan **30 días**. Si vuelves a Business en ese plazo, se reactiva con todo. Pasados los 30 días se borran el contenedor y los datos.
- **Desactivar a mano** (interruptor **AgentOffice activado**) para el AgentOffice pero no borra el volumen.
- **Borrado anticipado**: la cuenta no tiene un botón para ello; pídelo al soporte de Hilvana indicando tu organización.
- Quitar una clave de IA o un repo (**Quitar**) elimina esa credencial guardada.

<a id="faq"></a>
## Preguntas frecuentes

**¿Quién paga la IA?** Tu empresa: los agentes usan tu token o clave, así que el consumo va a tu suscripción de Claude o a tu cuenta de API. Hilvana no lo cobra ni lo reembolsa.

**¿Qué pasa si caduca o se revoca el token de IA?** Los agentes dejan de poder trabajar y las tareas fallan. Genera uno nuevo (`claude setup-token` o una clave de API), pégalo en **🔑 IA del AgentOffice** y pulsa **Guardar**: sustituye al anterior.

**¿Las ramas que crean los agentes llegan a mi repositorio?** Trabajan en una rama por tarea y, con un token con permiso de escritura, **empujan esa rama al remoto**; la fusión pasa por tu revisión. Sin permiso de escritura no podrán subirla.

**¿Hay escritorio o captura de pantalla en el modo nube?** No: el AgentOffice en la nube no tiene escritorio, así que no hay ventanas de escritorio ni capturas de pantalla reales; trabajan con el código y la terminal.

**¿Cómo roto un token de repo?** Crea uno nuevo en GitHub o GitLab con los mismos permisos mínimos, en **📦 Repos** añade de nuevo la URL con el token nuevo (**Añadir**; si hay dos del mismo servidor se usa el más reciente), quita el antiguo con **Quitar** y revoca el viejo en el proveedor.

**¿Por qué no puedo activarlo?** Mira [Errores típicos](#errores): plan no Business, cupo agotado o no eres owner/admin.
