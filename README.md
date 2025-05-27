# txflwtst
# Descripción
Este proyecto contiene un workflow de n8n que automatiza el flujo de trabajo tras una discovery call con un cliente potencial. A partir de la reserva de la llamada en correo, registra la cuenta (ver sheet_sample.xlsx) y crea una carpeta en drive para la cuenta. Una vez se tenga una transcripción de la llamada (ver transcripts_sample.txt), la procesa con un LLM (ver prompt_template.txt y LLM_outputs) para generar un informe (ver informes.txt), determina si es Qualified y crea un borrador de correo (ver email_drafts.txt) para notificar (y si es el caso, solicitar más información) al cliente.

# Prerrequisitos
- n8n

Cuentas y credenciales activas en n8n para:
- Email (para recibir notificaciones).
- Google Drive, Google Sheets, y Google Docs.
- API Gemini (para procesamiento de lenguaje).

# Instalación y Configuración
- Importar el workflow en n8n.
- Configurar las credenciales, verificando que esté asociada la credencial correcta.

# Ejecución
Paso 1:
- Al enviar un email de confirmación de Calendly, se activa el nodo Recibe correo (Outlook Trigger). 

Paso 2 y 3:
- Al proveer una transcripción de una llamada, se activa el procesamiento, la clasificación y la redacción del borrador de correo.

# Lógica del Workflow
Paso 1:
- Nodo "Recibe correo" captura nuevos emails.
- Text Classifier detecta notificaciones de Calendly.
- Information Extractor obtiene nombre, email y compañía.
- Google Drive → Create Folder crea una carpeta nombrada con el lead.
- Google Sheets → Append Row añade una fila con datos del lead y el enlace al folder.

Paso 2 y 3:

- Recibe transcripción de la llamada.
- Procesa info con un LLM prompteado para resumen y calificación a través de las preguntas provistas.
- Ex_Inf extrae JSON con background, needs, challenges, overall_Decision, engagement_Type, justification, email, lead_name, date.
- Busca info en sheets para recuperar la carpeta destino.
- Google Docs → Create Document y Update Document insertan el informe con el análisis.
- IF evalúa overall_Decision.
    - Qualified:
        - Actualiza sheets con Status (Call Had), Qualification (Qualified), Date y Engagement type.
        - Redacta correo según el contexto de la transcripción, la decisión y la justificación.
        - Gmail → Create Draft crea un borrador en Gmail.
    - Not Qualified:
        - Actualiza sheets con Status (Call Had), Qualification (Not Qualified) y Date.
        - Redacta correo según el contexto de la transcripción, la decisión y la justificación.
        - Gmail → Create Draft crea borrador en Gmail.

# Pruebas y Validación

- En transcripts_sample encuentra 4 casos (2 Qualified y 2 Not Qualified) para simular la etapa de transcripción, así como correos específicos.
- Use los mismos correos en la programación en Calendly y en la transcripción dada, para que los datos se actualicen en el archivo de google sheets asignado.
- Primero, programe una reunión por medio de Calendly.
- Verifique que se ha agregado la cuenta correspondiente a la reunión programada en el archivo de google sheets, además de la carpeta de la cuenta.
- Después, dé la transcripción de la llamada.
- Verifique que se creó un informe en la carpeta de la cuenta, que se actualizaron los datos en el archivo de google sheets y que se creó un borrador en el correo.

# Walkthrough video
Véase el video de la demostración en enlace_walkthrough_video.txt o siga directamente este enlace: https://youtu.be/GGt9PbAwEKo
