# Glosario del dominio FinOps

A continuación se recoge el vocabulario común del dominio, que contiene cualquier concepto cuya definición para el TFG es diferente a la normal o que es técnico y específico para este tema.

| Término | Definición |
| :--- | :--- |
| **Organización** | Empresa cliente que contrata y utiliza la plataforma. Es la unidad raíz del sistema: todos los datos quedan aislados por organización, garantizando que la información de un cliente nunca es visible para otro. |
| **Usuario** | Persona que interactúa con la plataforma. Pertenece a una organización y tiene uno de dos roles: administrador, con capacidad para configurar y lanzar misiones, o usuario regular, con acceso de solo lectura. |
| **Agente de IA** | Agente de Inteligencia Artificial descubierto y registrado en la plataforma a partir de los servicios cloud de la organización. En AWS corresponde a un *Amazon Bedrock Agent*. Véase también *Trabajador Digital*. |
| **Evento de Permiso** | Entrada inmutable del registro de permisos que se crea cada vez que cambian los roles IAM asociados a una misión. Registra la acción realizada, el rol afectado y si fue automática o manual. |
| **Configuración LLM** | Configuración del proveedor de inteligencia artificial de una organización. Define qué modelo se usa para cada rol funcional (*razonamiento* o *conversación*) del CAIO Virtual. |
| **Registro de Auditoría** | Traza inmutable de las operaciones relevantes realizadas en la plataforma. Es de solo escritura: nunca se modifica ni elimina, garantizando una pista de auditoría fiable para cumplimiento normativo. |
| **CAIO Virtual** | Chief AI Officer Virtual. El agente de IA de Theia Officer que actúa como trabajador digital encargado de la gobernanza y el control de costes de IA en la organización. |
| **Misión FinOps** | Tarea autónoma ejecutada por el CAIO Virtual para realizar operaciones de control financiero sobre los agentes de IA en la nube. |
| **Trabajador Digital** | Denominación en la interfaz de usuario del producto para referirse a un agente de IA descubierto en un proveedor cloud (equivale a *Agente de IA* en este modelo). |
| **Credencial AWS** | Par de claves (access key ID + secret access key) de AWS que permiten al sistema acceder a los servicios de Amazon en nombre de la organización. |
| **Introspección** | Diagnóstico automatizado que evalúa el estado de las credenciales AWS de una organización: verifica la identidad, los permisos IAM disponibles y el acceso a los servicios de Bedrock. |
| **Amazon Bedrock** | Servicio de AWS que permite desplegar y orquestar agentes de IA usando modelos fundacionales de terceros (como Anthropic Claude). |
| **Amazon Bedrock Agent** | Un agente de IA desplegado en Amazon Bedrock. Es la entidad que se descubre, monitoriza y cuyo coste se analiza. |
| **AWS Cost Explorer** | Servicio de AWS que proporciona visibilidad sobre el gasto en la nube, con capacidad de filtrar por servicio, recurso y periodo de tiempo. |
| **UnblendedCost** | Métrica de AWS Cost Explorer que representa el coste real de un recurso sin mezclar descuentos de otros recursos ni reservas compartidas. Es la métrica más adecuada para atribuir costes exactos a agentes individuales. |
| **IAM** | Identity and Access Management. Sistema de gestión de identidades y accesos de AWS que controla qué operaciones puede realizar cada entidad. |
| **Rol temporal de misión** | Rol IAM que se adjunta a las credenciales de la organización únicamente durante la ejecución de una misión y se elimina automáticamente al terminar. Implementa el principio de mínimo privilegio. |
| **Principio de mínimo privilegio** | Práctica de seguridad según la cual una entidad solo debe tener los permisos estrictamente necesarios para realizar su tarea, y solo durante el tiempo en que la necesita. |
| **Dato de Facturación** | Registro del coste real de un agente de IA en AWS durante un periodo determinado, obtenido directamente de AWS Cost Explorer. |
| **Registro de Uso de LLM** | Log de cada llamada a un modelo de lenguaje grande realizada por la propia plataforma, con el número de *tokens* consumidos y su coste estimado. |
| **Token** | Unidad de procesamiento de los modelos de lenguaje. Aproximadamente equivale a 3/4 de una palabra en inglés. Las APIs de LLM facturan por tokens de entrada (*input tokens*) y tokens de salida (*output tokens*). |
| **Coste de orquestación** | El coste total en *tokens* que genera una única petición de usuario a un agente de IA agéntico, incluyendo todas las llamadas internas al LLM que el agente realiza en el proceso de razonamiento y ejecución de herramientas. |
| **Umbral de gasto** | Límite de coste definido por el administrador para un agente o periodo. Cuando el gasto real supera el umbral, el sistema genera una alerta automática. Es el concepto central de CU-13. |
| **Decisión Estratégica FinOps (F)** | Cada una de las 11 preguntas de gobernanza financiera de IA que el framework propio de Theia Officer evalúa para una organización (F1 a F11). |
| **Action Group** | Término de Amazon Bedrock para un conjunto de funciones (AWS Lambda) que un agente puede invocar durante su ejecución. Equivalente al concepto de *Tool Calling* en otros frameworks de IA agéntica. |