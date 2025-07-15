# Bot de Asistencia con n8n y Chatwoot (Configuración Básica)

Este repositorio contiene la lógica de automatización en n8n para un bot de asistencia integrado con Chatwoot. Este setup está diseñado para gestionar conversaciones, responder a preguntas y escalar a agentes humanos cuando sea necesario, utilizando las funcionalidades nativas de Chatwoot para la gestión del estado de la conversación y sin dependencias de bases de datos externas como Redis o PostgreSQL.

## 🚀 Puesta en Marcha Rápida

Sigue estos pasos para desplegar y configurar tu bot de asistencia en poco tiempo.

### 1. Configuración de la API de Evolución (Evolution API)

Asegúrate de que tu instancia de Evolution API esté configurada para interactuar con Chatwoot.

* **Variables de Entorno:**
  * Verifica que la variable `CHATWOOT_ENABLED` esté configurada como `true` en tu entorno de Evolution API.

* **Integración con Chatwoot:**
  * Necesitarás los siguientes datos de tu instancia de Chatwoot para la configuración en Evolution API:
    * `account id` de Chatwoot: El ID de tu cuenta en Chatwoot.
    * `Token` de la cuenta de Chatwoot: Tu token de acceso a la API de Chatwoot.
    * `Nombre del inbox`: El nombre que deseas darle a tu bandeja de entrada de Chatwoot que gestionará el bot (ej. "Soporte Bot").
  * **Habilitar todas las opciones y auto-creación:** Asegúrate de que, durante la configuración de la integración de Chatwoot en Evolution API, habilites todas las opciones pertinentes, incluida la auto-creación de contactos/conversaciones si está disponible.

* **Crear Bandeja de Entrada:**
  * Desde la interfaz o las funcionalidades de Evolution API, crea la bandeja de entrada de Chatwoot usando los datos proporcionados. Esto vinculará Evolution API con tu Chatwoot.

### 2. Configuración en Chatwoot

Una vez que Evolution API esté enlazado, configura Chatwoot para que envíe los eventos relevantes a tu flujo de n8n.

* **Webhook de Entrada:**
  * En Chatwoot, ve a `Configuración` -> `Inboxes` -> (Tu Inbox) -> `Configuración` -> `Webhooks`.
  * Crea un nuevo Webhook.
  * **URL del Webhook:** Esta será la URL de tu Webhook en n8n (la del nodo `chatwoot` en tu flujo).
  * **Eventos:** Selecciona al menos el evento `Mensaje creado (message_created)`. Esto asegura que tu bot recibe cada nuevo mensaje.

* **Filtro de Conversación:**
  * Para optimizar el rendimiento y evitar que el bot responda a conversaciones ya gestionadas por humanos, tu flujo de n8n debe filtrar las conversaciones.
  * **Configuración en n8n:** Dentro de tu workflow de n8n, después del nodo `chatwoot` (el webhook de entrada), asegúrate de tener un nodo `Filter` (o lógica condicional) que procese mensajes **solo cuando el `estado` de la conversación es `pending`**. Esto es crucial para la lógica de escalada a humano.

### 3. Configuración del Flujo de n8n

Tu flujo de n8n (`.json` del workflow) debe estar configurado para:

* **Recibir el Webhook de Chatwoot:** Un nodo `Webhook` configurado con la URL que pondrás en Chatwoot.

* **Filtrar Mensajes:** Un nodo `Filter` para procesar solo los mensajes entrantes de contactos y con estado `pending`.

* **Mapear Constantes (`const` Node):** Un nodo `Set` (que usas como `const`) para extraer y normalizar datos clave de la entrada de Chatwoot (IDs de cuenta, conversación, nombre/número de usuario, etc.) para que estén disponibles para el agente de IA y las tools.

* **Agente de IA (`WP agent`):** Configurado con un `system prompt` que define su personalidad, reglas de interacción y, crucialmente, las reglas de uso de herramientas para la escalada a humano.

* **Herramientas (Tools):**
  * `talk_human`: Una tool (`HTTP Request Tool`) que se conecta a la API de Chatwoot para cambiar el estado de la conversación a `open` o asignarla a un agente/equipo, indicando que se necesita intervención humana. Asegúrate de que esta tool recibe los parámetros (`account_id`, `conversation_id`) correctamente desde el agente o el contexto del flujo.
  * `send_msg_admins`: Otra tool (`HTTP Request Tool`) que envía una notificación a tus administradores a través de Evolution API (o el servicio de mensajería que uses) cuando una conversación ha sido escalada. Asegúrate de que recibe los parámetros del usuario (`user_name`, `user_phone`, `conv_id`, `last_message`) para el mensaje de notificación.

## 🤝 Contribución

Si encuentras mejoras o tienes sugerencias para este flujo básico, no dudes en abrir un *issue* o enviar un *pull request*.
