# 16 · AgentOffice en la nube (Business)

> Revisado para Hilvana **6.8.0** · Texto de la página de tu cuenta ▸ **🏢 AgentOffice en la nube**. Sin capturas: la página de la cuenta no forma parte de la aplicación local.

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
