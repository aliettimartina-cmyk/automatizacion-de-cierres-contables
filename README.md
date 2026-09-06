Documentación del checkpoint 4
# Checkpoint 4 — Integraciones Avanzadas e Interconexión de Sistemas

## Archivos de este checkpoint

- **checkpoint4_Alietti_Martina.json** — Workflow principal: "Ingesta Sumas y Saldos - Email". Contiene el Gmail Trigger, la lógica de seguridad (lista blanca de remitentes + filtro anti-auto-respuesta), el Look-up de duplicados contra Airtable, la extracción del Excel adjunto, y la delegación del procesamiento al Manager vía Execute Workflow con espera síncrona.

- **checkpoint4_2_Alietti_Martina.json** — Workflow "Manager_Proyecto Integrador" (ya existente de checkpoints anteriores, ampliado en este). Recibe la delegación del workflow de ingesta a través de un segundo trigger ("When Executed by Another Workflow"), y agrega en esta entrega el tercer conector (Google Drive) y el cambio de notificación final de envío directo a "Create Draft".

## Tercer conector: Google Drive (no HubSpot/Calendar)

Se eligió Google Drive como tercer conector, en lugar de un CRM o Calendar, porque el caso de uso real de esta herramienta (automatización de cierres contables para un equipo interno) no involucra gestión de clientes ni eventos de calendario. Drive resuelve una necesidad genuina del proceso: archivar automáticamente cada cierre procesado como respaldo de auditoría, en un archivo de texto (`Cierre_<periodo>.txt`) con el detalle completo de la validación de partida doble y las cuentas procesadas.

## Desvío documentado: ubicación del nodo IF de seguridad

La consigna especifica colocar el nodo IF "inmediatamente posterior al trigger de entrada de la casilla de correo corporativa". En esta implementación, el IF de seguridad ("If_Remitente Autorizado") está ubicado después de un nodo de búsqueda en Airtable ("Search records_Buscar Remitente Autorizado"), no inmediatamente después del Gmail Trigger.

**Motivo:** la condición de seguridad no puede evaluarse únicamente con los datos que trae el evento del trigger — necesita confirmar si el remitente del mail figura en la tabla "Equipo_Autorizado" de Airtable, un dato externo que debe consultarse antes de poder decidir. El nodo de búsqueda es, por lo tanto, un prerrequisito de datos para la condición, no un paso de procesamiento que debería ir después de la validación.

El filtro de asunto (auto-respuestas, mails autoenviados por el propio sistema) sí podría evaluarse con los datos crudos del trigger sin necesitar a Airtable, y se combinó en el mismo nodo IF junto con la validación de remitente por eficiencia — ambas condiciones se resuelven en una sola decisión antes de continuar el procesamiento.

## Scopes de Gmail: mínimo privilegio documentado

La credencial de Gmail utilizada en producción tiene actualmente el scope amplio `https://mail.google.com/` (control total sobre la cuenta). Se identificó y documentó (ver captura `checkpoint4_scopes_minimo_privilegio_Alietti_Martina.png`) que el sistema solo requiere `gmail.compose` (para crear los borradores) y `gmail.readonly` (para leer los mails entrantes y sus adjuntos) — nunca envía, borra ni modifica correos existentes.

Se decidió no ejecutar el cambio de scopes sobre la credencial en producción, para no interrumpir el flujo ya validado con pruebas reales de punta a punta durante el desarrollo de este checkpoint. Queda como mejora identificada para una próxima iteración.
