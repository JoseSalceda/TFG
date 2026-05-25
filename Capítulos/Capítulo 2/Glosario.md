# Glosario del dominio FinOps

A continuación se recoge el vocabulario común del dominio, que contiene cualquier concepto cuya definición para el TFG es diferente a la normal o que es técnico y específico para este tema.

| Término | Definición |
| :--- | :--- |
| **Organización** | Empresa cliente que contrata y utiliza la plataforma. Es la unidad raíz del sistema: todos los datos quedan aislados por organización, garantizando que la información de un cliente nunca es visible para otro. |
| **Usuario** | Persona que interactúa con la plataforma. Pertenece a una organización y tiene uno de dos roles: administrador, con capacidad para configurar y lanzar misiones, o usuario regular, con acceso de consulta y simulación no mutante según la operación. |
| **Agente de IA** | Agente de Inteligencia Artificial descubierto y registrado en la plataforma a partir de los servicios cloud de la organización. En AWS puede corresponder a un *Amazon Bedrock Agent*. Véase también *Trabajador Digital*. |
| **Evento de Permiso** | Entrada del registro de permisos usada por la plataforma base cuando una misión autónoma adjunta o retira roles IAM. En este TFG aparece solo como contexto de seguridad; las misiones FinOps CU-07 a CU-12 se trazan mediante estado de misión de usuario y auditoría. |
| **Configuración LLM** | Configuración del proveedor de inteligencia artificial de una organización. Define qué modelo se usa para cada rol funcional (*razonamiento* o *conversación*) del CAIO Virtual. |
| **Registro de Auditoría** | Traza de las operaciones relevantes realizadas en la plataforma. Se usa para dejar evidencia revisable de sincronizaciones, análisis y acciones administrativas. |
| **CAIO Virtual** | Agente de IA de la plataforma que actúa como trabajador digital encargado de la gobernanza y el control de costes de IA en la organización. |
| **Misión FinOps** | Tarea guiada del catálogo FinOps que orienta al usuario en una comprobación o acción de control financiero. Conserva progreso, pasos y evidencia de revisión, pero no equivale a una misión autónoma de gestión de permisos IAM. |
| **Trabajador Digital** | Denominación en la interfaz de usuario del producto para referirse a un agente de IA descubierto en un proveedor cloud (equivale a *Agente de IA* en este modelo). |
| **Credencial AWS** | Par de claves (access key ID + secret access key) de AWS que permiten al sistema acceder a los servicios de Amazon en nombre de la organización. |
| **Introspección** | Diagnóstico automatizado que evalúa el estado de las credenciales AWS de una organización: verifica la identidad, los permisos IAM disponibles y el acceso a los servicios de Bedrock. |
| **Amazon Bedrock** | Servicio de AWS que permite desplegar y orquestar agentes de IA usando modelos fundacionales de terceros (como Anthropic Claude). |
| **Amazon Bedrock Agent** | Un agente de IA desplegado en Amazon Bedrock. Es la entidad que se descubre y sobre la que pueden aplicarse acciones controladas como pruebas RAG o prompt caching cuando la configuración lo permite. |
| **AWS Cost Explorer** | Servicio de AWS que proporciona visibilidad histórica sobre el gasto en la nube, con capacidad de filtrar por servicio, periodo y dimensiones soportadas. No ofrece por sí solo detalle de cada invocación o token. |
| **UnblendedCost** | Métrica de AWS Cost Explorer que representa coste sin mezclar ciertos descuentos o reservas compartidas. En este TFG se utiliza como base de agregación de coste, no como garantía de atribución exacta por agente. |
| **Cost allocation tag** | Etiqueta de asignación de coste que AWS puede exponer en Cost Explorer cuando está activada y existe uso posterior. En este TFG se usa `theia-agent-id` como clave de atribución preparada por la organización. |
| **IAM** | Identity and Access Management. Sistema de gestión de identidades y accesos de AWS que controla qué operaciones puede realizar cada entidad. |
| **Rol temporal de misión** | Rol IAM que puede adjuntarse durante la ejecución de misiones autónomas de infraestructura y retirarse al terminar cuando el caso lo requiere. No es el mecanismo principal de las misiones FinOps implementadas en CU-07 a CU-12. |
| **Principio de mínimo privilegio** | Práctica de seguridad según la cual una entidad solo debe tener los permisos estrictamente necesarios para realizar su tarea, y solo durante el tiempo en que la necesita. |
| **Dato de Facturación** | Registro de coste de IA en AWS durante un periodo determinado, obtenido de AWS Cost Explorer. Incluye proveedor, recurso, periodo, importe, moneda y servicio; puede estar asociado a un agente si existe una señal de atribución fiable o quedar como coste no asociado. |
| **Coste no asociado** | Parte del gasto real de Amazon Bedrock que no puede vincularse a un agente concreto con la información disponible. El sistema lo conserva separado para no ocultar gasto ni inventar atribuciones. |
| **Registro de Uso de LLM** | Log de cada llamada a un modelo de lenguaje grande realizada por la propia plataforma, con el número de *tokens* consumidos y su coste estimado. |
| **Token** | Unidad de procesamiento de los modelos de lenguaje. Aproximadamente equivale a 3/4 de una palabra en inglés. Los proveedores LLM suelen facturar por tokens de entrada (*input tokens*) y tokens de salida (*output tokens*). |
| **Coste de orquestación** | El coste total en *tokens* que genera una única petición de usuario a un agente de IA agéntico, incluyendo todas las llamadas internas al LLM que el agente realiza en el proceso de razonamiento y ejecución de herramientas. |
| **Umbral de gasto** | Límite de coste definido por el administrador para un agente o periodo. Cuando el gasto real supera el umbral, el sistema genera una alerta automática. Es el concepto central del futuro CU-14; CU-13 queda reservado para proyección de tendencia. |
| **RAG** | *Retrieval Augmented Generation*. Técnica que permite a un agente recuperar fragmentos de conocimiento antes de responder. En CU-11 se optimiza de forma controlada el número de fragmentos recuperados. |
| **Knowledge Base** | Base de conocimiento asociada a un agente Bedrock. Proporciona el contexto recuperado por RAG y condiciona si CU-11 es aplicable. |
| **Prompt caching** | Mecanismo que puede reutilizar partes repetidas de un prompt para reducir coste o latencia en condiciones concretas. Su beneficio depende de modelo, región, TTL, prefijo cacheable y tráfico repetido. |
| **Decisión Estratégica FinOps (F)** | Cada una de las 11 preguntas de gobernanza financiera de IA que el framework propio de la solución evalúa para una organización (F1 a F11). |
| **Action Group** | Término de Amazon Bedrock para un conjunto de funciones (AWS Lambda) que un agente puede invocar durante su ejecución. Equivalente al concepto de *Tool Calling* en otros frameworks de IA agéntica. |
