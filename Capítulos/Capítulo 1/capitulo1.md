# 1 Introducción

## 1.1 Contexto del proyecto

Este TFG busca diseñar, implementar y validar un módulo de gestión financiera en la nube para agentes de inteligencia artificial. El trabajo se desarrolla en colaboración con Theia Craft, empresa en la que nace la necesidad de incorporar capacidades FinOps dentro de una plataforma de gobierno de IA. La solución propone un CAIO Virtual capaz de actuar como trabajador digital de una organización, acompañando tareas de gobierno, automatización y toma de decisiones. En ese contexto, el TFG se centra en una parte concreta del producto: la Misión FinOps.

## 1.2 Problema identificado

Muchas organizaciones, incluso las más grandes, siguen sin tener claro cómo adoptar una IA que avanza tan rápido que cuesta mantenerse al día con sus avances. Estas empresas saben de la utilidad de la IA y cómo usarla de forma correcta puede hacerla una herramienta valiosísima para su negocio. Uno de los muchos desafíos, y que constituye el núcleo central de este proyecto, es la gestión de costes. El objetivo principal de las organizaciones es maximizar su rentabilidad, por lo que optimizar cada euro es clave. El problema es que el gasto puede repartirse entre servicios, regiones, equipos y proveedores, y no siempre resulta evidente dónde se produce, cómo evoluciona o si aporta valor suficiente [1].

En el ámbito de la IA y la nube, los costes asociados a las peticiones de LLMs y al despliegue de agentes pueden comportarse como una caja negra: infraestructura, bases de datos vectoriales, contexto recuperado y *tokens* consumidos no siempre aparecen unidos en una misma vista. Las principales plataformas cloud (AWS, GCP, Azure) ofrecen herramientas para gestionar costes, pero la atribución fina a agentes, sesiones o funcionalidades suele requerir configuración adicional, exportaciones, etiquetas o telemetría propia. En este contexto, la disciplina corporativa de FinOps aplicada a la Inteligencia Artificial se convierte en una necesidad inmediata, ya que la rentabilidad y el control del gasto son pilares fundamentales para la sostenibilidad de cualquier empresa.

## 1.3 Solución propuesta

Para dar respuesta a esta problemática, el presente TFG propone el diseño y desarrollo de varias Misiones FinOps integradas dentro del ecosistema de agentes virtuales de Theia Craft. En lugar de depender únicamente de paneles de control estáticos (*dashboards*) que requieren la revisión manual de un ingeniero, la solución implementada expone un conjunto de servicios backend y flujos de misión que consultan AWS Cost Explorer, normalizan datos de coste de Amazon Bedrock, conservan el gasto no asociado y presentan los resultados al usuario con trazabilidad y soporte de auditoría.

Las misiones FinOps se organizan en dos grupos. Primero, misiones de visibilidad de costes: coste total AWS Bedrock, coste por agente, anomalías y coste LLM estimado de la propia plataforma. Segundo, misiones de optimización controlada: ajuste de recuperación RAG y prompt caching en agentes Bedrock cuando las condiciones técnicas lo permiten. La solución no promete ahorro automático; proporciona evidencia, separa estimaciones de facturación oficial y deja las acciones sensibles bajo control administrativo.

# 2 Marco teórico

## 2.1 Estado del arte

### 2.1.1 Contexto Theia Craft

Theia Craft es una consultoría informática especializada en B2B, a diferencia del modelo B2C tradicional. Es una *startup* con apenas un año de vida fundada por Kiyoshi Omaza, ingeniero con mucha experiencia en grandes empresas tecnológicas de Madrid y tutor en este TFG. Actualmente Theia Craft está centrada en el desarrollo de tecnologías para ayudar a empresas a mejorar la toma de decisiones a través de una propuesta que combina investigación del funcionamiento de la normativa, de la empresa en la que se implementa y que está potenciada por el uso de varias tecnologías. Entre ellas destaca la IA mediante el uso de trabajadores digitales que disparan la productividad de los empleados, ya que así se pueden centrar en las tareas que verdaderamente importan.

Una de las peticiones más repetidas por los distintos clientes interesados por la propuesta de Theia Craft es la de aportar mayor visibilidad y control sobre las finanzas de la empresa, ya que con el paso de los años se han visto obligados a usar varias herramientas cloud a la vez. En algunos contextos empresariales conviven varios proveedores cloud, cada uno con sus propias herramientas y formas de gestionar los costes. Esta fragmentación, sumada a la adopción acelerada de asistentes de IA y flujos agénticos, ha provocado que el coste de la inferencia (las llamadas a LLMs) se vuelva impredecible. La falta de herramientas unificadas que crucen el nivel táctico (consumo de *tokens* y contexto) con el nivel estratégico (facturación y retorno de inversión) hizo que Theia Craft viera la necesidad de investigar y desarrollar una solución FinOps nativa e integrada en su ecosistema.


### 2.1.2 Soluciones existentes

En la actualidad, el reto de la gestión de costes de IA se está abordando desde tres ángulos principales, cada uno con fortalezas y limitaciones para un entorno de IA Agéntica:

1. Herramientas nativas Cloud (Cloud Cost Management): Plataformas como AWS Cost Explorer, GCP Cloud Billing y Azure Cost Management han comenzado a incorporar funcionalidades específicas para recursos de *machine learning* [1]. Proporcionan un buen control a nivel macroestructural, pero la atribución a interacciones individuales, misiones de un agente o consumo de *tokens* requiere configuración adicional, etiquetas, exportaciones o telemetría complementaria.

2. Plataformas de Observabilidad de LLMs (LLMOps): Herramientas de terceros como LangSmith, Helicone o Datadog LLM Observability se centran en el nivel micro. Capturan cada petición, la longitud del *prompt*, el rendimiento y los *tokens* consumidos, calculando un coste estimado [2]. Sin embargo, a menudo su facturación es estimada y no cruza directamente con la factura real de la nube del cliente, separando la telemetría operativa de la realidad financiera corporativa.

3. Plataformas FinOps especializadas: Soluciones SaaS de gestión en la nube (ej. Finout, Ternary, CloudZero) están evolucionando para incluir *unit economics* de IA [3] [4]. Permiten asignar costes complejos uniendo logs y facturas. El inconveniente radica en que son herramientas pasivas o de reporte (*dashboards*), pensadas para ser consumidas por un equipo financiero o un ingeniero. Exigen la exportación de datos a plataformas externas y no aprovechan la proactividad del agente de IA para actuar sobre esos datos.

El vacío en el estado del arte actual reside en la IA Agéntica aplicada a FinOps. Mientras las soluciones existentes se enfocan en otorgar visibilidad para que un humano tome las decisiones, la propuesta de este TFG busca integrar la observabilidad financiera directamente en el flujo de trabajo del Agente Virtual comercializado por Theia Craft, permitiendo que el sistema guíe la comprobación, sincronización y revisión de costes sin ocultar las limitaciones de la fuente de datos.

### 2.1.3 Aplicaciones empresariales de IA

La Inteligencia Artificial ha pasado de ser una herramienta de experimentación a un componente estructural en las operaciones empresariales. Según proyecciones de consultoras tecnológicas como Gartner, para finales de 2026, el 40% de las aplicaciones empresariales integrarán agentes de IA específicos para tareas, frente a menos del 5% que existía en 2025 [5]. Las empresas utilizan la IA para la atención al cliente automatizada, la generación de código, el análisis masivo de datos no estructurados y la redacción de informes. Sin embargo, esta adopción masiva ha introducido el reto de la escalabilidad financiera, haciendo indispensable el control del retorno de inversión (ROI).

### 2.1.4 Capacidades actuales de la IA 

Las capacidades actuales de la IA están fuertemente marcadas por el aumento de las ventanas de contexto (capaces de procesar millones de *tokens*, equivalente a libros enteros en una sola petición) y la multimodalidad (procesamiento simultáneo de texto, imagen y audio). No obstante, estas capacidades conllevan un coste computacional lineal. Procesar un documento PDF de 400 páginas en cada interacción garantiza un contexto excelente, pero dispara el número de *Input Tokens*, lo que subraya la necesidad de prácticas como el *Prompt Caching* o el enrutamiento dinámico de modelos (*Model Routing*) para contener el gasto [6].

### 2.1.5 Agentes de IA, ¿qué son?

En el ámbito de la Inteligencia Artificial moderna, un agente se define como un sistema de software autónomo que utiliza un Modelo de Lenguaje Grande (LLM) como su motor central de razonamiento para lograr un objetivo específico. A diferencia de un modelo tradicional que se limita a procesar texto, un agente de IA está dotado de cuatro componentes arquitectónicos fundamentales:

- Perfil/Objetivo: Un propósito o rol definido (ej. "Eres un agente de recursos humanos").
- Memoria: Capacidad para retener el contexto a corto plazo (el historial de la conversación actual) y a largo plazo (acceso a bases de datos vectoriales).
- Planificación: La habilidad de descomponer un problema complejo en una secuencia de pasos lógicos y manejables (usando paradigmas como *Chain of Thought*) [7].
- Herramientas (*Tool Calling*): La capacidad crítica de actuar sobre su entorno. El agente puede decidir cuándo invocar APIs externas, ejecutar código, consultar bases de datos o, en el caso de este TFG, llamar a los servicios de facturación de AWS para obtener datos reales.

En definitiva, los agentes actúan como trabajadores digitales (*Agentic Workflows*) que pueden llevar a cabo flujos de trabajo de principio a fin, interactuando con el entorno corporativo y solicitando intervención humana únicamente para validaciones críticas o permisos (acciones semiautomáticas).


### 2.1.6 Tipos de agentes de IA: Generativa (Tradicional) vs Agéntica

- IA Generativa (GenAI tradicional): Se basa en sistemas puramente reactivos y de un solo turno (*single-turn*). El usuario introduce una instrucción o *prompt* (entrada) y el modelo genera directamente una respuesta (salida). El flujo es lineal, como ocurre en un asistente de redacción clásico. Desde el punto de vista financiero (FinOps), su coste es altamente predecible: la empresa paga exclusivamente por los *tokens* enviados en la petición y los *tokens* generados en la respuesta durante esa interacción única.

- IA Agéntica (*Agentic AI*): Representa un paradigma proactivo e iterativo. Ante una única petición del usuario, el sistema entra en un bucle autónomo de razonamiento y acción (arquitectura ReAct [8]). Para resolver la tarea, el agente puede "hablar" consigo mismo e invocar al LLM subyacente múltiples veces en la sombra (por ejemplo: primero piensa qué hacer, luego llama a una base de datos, después lee el resultado, detecta un error, vuelve a consultar la base de datos y finalmente redacta la solución). Financieramente, este comportamiento convierte el coste en una variable impredecible. Una sola petición del usuario puede multiplicar el consumo de *tokens* (conocido como coste de orquestación), haciendo que una interacción pase de costar fracciones de céntimo a varios dólares sin que el usuario sea consciente. Esta diferencia fundamental entre la IA Generativa (coste por petición) y la IA Agéntica (coste por bucle de razonamiento) es el principal detonante que justifica la necesidad inmediata de integrar herramientas de observabilidad FinOps microscópicas como la desarrollada en este TFG [9].


### 2.1.7 Plataformas Cloud

El despliegue de soluciones de IA empresarial requiere de una infraestructura robusta, dominada por los tres grandes proveedores de nube pública, los cuales ofrecen servicios gestionados de IA:

### 2.1.7.1 AWS

Amazon Web Services centraliza su oferta de IA generativa en Amazon Bedrock. Permite el despliegue de "Bedrock Agents" utilizando modelos fundacionales corporativos (ej. Anthropic Claude, Amazon Titan).

Los costes se gestionan a nivel macro mediante AWS Cost Explorer. Para obtener visibilidad de invocaciones, *tokens* o sesiones concretas es necesario apoyarse en mecanismos adicionales, como logs de invocación, perfiles de inferencia, etiquetas de asignación de coste o telemetría propia. Por tanto, Cost Explorer es adecuado para coste histórico y agregado, pero no sustituye por sí solo una observabilidad LLM completa.

De las tres opciones evaluadas, AWS se selecciona como alternativa principal para este TFG por su encaje con Amazon Bedrock, Cost Explorer, IAM y los mecanismos de observabilidad de costes que necesita el prototipo funcional.

### 2.1.7.2 GCP

Google Cloud Platform ofrece Vertex AI para modelos generativos y Dialogflow CX para agentes conversacionales robustos. La facturación se consolida en Cloud Billing y se puede exportar a BigQuery para un análisis detallado (Standard usage cost export).

Aunque GCP permite un análisis granular cruzando BigQuery con los logs de operaciones de los agentes, requiere conocimiento avanzado de SQL y una configuración manual de permisos y exportaciones para correlacionar los costes de infraestructura con el uso específico de los LLMs.

Es una plataforma bastante robusta y con gran número de opciones, pero se percibe que todavía está algo lejos de su potencial definitivo.

### 2.1.7.3 Azure

Microsoft Azure provee Azure OpenAI Service y Azure Machine Learning. Está profundamente integrado con su ecosistema enterprise, facturando por *tokens* procesados en sus *endpoints* online. Azure Cost Management ofrece herramientas para el etiquetado y presupuestos, pero al igual que sus competidores, cruzar el uso técnico diario con el gasto financiero corporativo (FinOps) de múltiples modelos y endpoints en diferentes suscripciones requiere soluciones de terceros o desarrollos a medida.

En el alcance de este TFG, Azure se considera una alternativa menos adecuada porque la integración analizada quedaba más ligada al ecosistema Azure y a requisitos de configuración específicos que no formaban parte del entorno principal de Theia Craft.

### 2.1.8 Plataforma utilizada

Las tres plataformas cloud mencionadas son las más grandes de la industria, cada una con sus ventajas y limitaciones. De ellas, la actividad de Theia Craft se concentra principalmente en AWS y GCP. En este TFG se utiliza AWS como plataforma de referencia porque Amazon Bedrock permite trabajar con agentes gestionados y porque Cost Explorer ofrece una API adecuada para obtener coste histórico de los servicios consumidos. La elección no implica descartar otros proveedores, sino acotar el alcance implementado para construir un prototipo funcional verificable.

### 2.1.9 Tabla 1. Comparativa de características FinOps.

| Característica FinOps (GenOps) | AWS | GCP | Azure |
| :--- | :--- | :--- | :--- |
| **Visibilidad de Costes (Macro)** | Sí | Sí | Sí |
| **Exportación a Motor Analítico SQL** | A medias | Nativo | A medias |
| **Visibilidad de Nivel Micro (*Tokens*)** | Parcial, requiere telemetría adicional | Parcial, requiere exportaciones/logs | Parcial, requiere telemetría adicional |
| **Detección de Anomalías de IA** | Parcial, con servicios de coste y lógica adicional | Parcial, con exportación y análisis | Parcial, con Cost Management y reglas |
| **Atribución de Coste por Agente** | Condicionada por etiquetas, perfiles o identidad | Condicionada por etiquetas/exportaciones | Condicionada por etiquetas y recursos |
| **Acceso API (*Tool Calling*)** | Sí | Sí | Sí |


## 2.2 Justificación de la propuesta

La gestión financiera en la nube (Cloud Financial Management) se ha basado históricamente en métricas predecibles, como el tiempo de actividad de un servidor o el almacenamiento consumido. Sin embargo, los Modelos de Lenguaje Grande (LLMs) introducen un paradigma de facturación dinámico basado en el consumo de *tokens*. Este modelo de precios fluctúa enormemente dependiendo de la longitud del contexto y de los bucles de razonamiento interno que realiza la IA, provocando que el rastreo manual de estas micro-transacciones a través de miles de interacciones corporativas sea inasumible para los equipos humanos [10].

Ante este cambio, el desarrollo de un módulo FinOps adaptado aborda el problema de raíz. Implementar misiones dedicadas a aportar visibilidad financiera da valor al ecosistema de Theia Craft porque estructura datos de coste, uso LLM y configuración de agentes dentro de una misma experiencia de gobierno. En lugar de depender únicamente de la revisión periódica de paneles externos por parte de un ingeniero, la arquitectura propuesta permite consultar datos estructurados, revisarlos desde la plataforma y dejar evidencia auditable de las operaciones relevantes [4].

Es fundamental dejar claro que este reto no puede resolverse utilizando Inteligencia Artificial generativa estándar. Un LLM aislado carece de acceso fiable a la infraestructura de facturación y no debe inventar cálculos económicos. Por este motivo, el proyecto separa la conversación del cálculo determinista: las misiones llaman a endpoints FastAPI que consultan Cost Explorer, guardan registros diarios de Amazon Bedrock y devuelven resultados estructurados. La precisión del resultado queda limitada por la disponibilidad y granularidad de AWS Billing, no por la generación del modelo.

El núcleo que aporta viabilidad práctica a este Trabajo de Fin de Grado es, precisamente, su integración técnica con datos reales de facturación. En el alcance implementado, la plataforma usa AWS Cost Explorer para obtener coste diario de Amazon Bedrock y conserva la atribución por agente cuando AWS la expone mediante etiquetas o mecanismos de atribución preparados previamente. El sistema no promete reconstruir cada sesión ni cada token desde Cost Explorer; esos datos requerirían una ampliación con logs de invocación, exportaciones CUR o telemetría específica.

Esta integración responde además a una necesidad estricta de trazabilidad. Las auditorías FinOps requieren relacionar el impacto macroeconómico de la factura con señales operativas como agentes, perfiles, fuentes de uso o registros internos. Gestionar este contexto entre distintos servicios de monitorización justifica el uso de un módulo integrado en el CAIO Virtual en lugar de un simple *script* de extracción de datos.
 
Por último, la selección de Amazon Bedrock como infraestructura base responde a los requisitos de seguridad y al contexto tecnológico de Theia Craft. Aunque el mercado ofrece frameworks de orquestación alternativos de código abierto (como LangChain o LangGraph), Bedrock proporciona capacidades nativas como agentes gestionados, perfiles de inferencia, integración con IAM y opciones de observabilidad. Esto facilita aplicar el principio de mínimo privilegio y mantener los datos de facturación dentro del entorno controlado de AWS [11]. Esta suma de capacidades hace de AWS un entorno adecuado para el alcance de la propuesta.


## 2.3 Solución propuesta

La solución propuesta actual es un "*Pipeline* de FinOps & Gobernanza" embebido en el producto de Theia Craft. Técnicamente consta de:

- Un modelo de datos unificado para representar "Misiones" e "Introspección de Sistemas", agnosticizando las complicaciones técnicas ligadas a la nube específica (AWS).
- Un flujo de usuario interactivo basado en misiones, donde el sistema guía al administrador o usuario regular para comprobar credenciales, sincronizar datos, revisar resultados y ejecutar acciones controladas cuando proceda.
- Servicios backend FastAPI que actúan como capa determinista entre la interfaz y AWS Cost Explorer. Esta capa sincroniza registros diarios, conserva coste no asociado, ejecuta un detector local de anomalías y escribe auditoría. Integraciones futuras podrían incorporar Lambda, logs de invocación o exportaciones CUR, pero no forman parte del núcleo implementado en este alcance.


## 2.4 Objetivos

Para dar respuesta a la problemática planteada y estructurar el desarrollo de la solución, se han definido un objetivo general y una serie de objetivos específicos que guiarán las distintas fases del proyecto.

### 2.4.1 Objetivo general

Diseñar, desarrollar y validar un módulo de gestión financiera en la nube, orientado a la Misión FinOps, para integrarlo en una plataforma de gobierno de IA como parte de un CAIO Virtual.

### 2.4.2 Objetivos específicos

Para alcanzar el objetivo general propuesto, el proyecto se desglosa en los siguientes objetivos específicos, los cuales se encuentran directamente mapeados con las fases de ingeniería del software y los capítulos de este documento:

1. Definir requisitos y modelo de dominio.
2. Definir análisis y diseño arquitectónico.
3. Desarrollar un prototipo funcional de la Misión FinOps.
4. Evaluar el prototipo en entorno controlado.


## 2.5 Estructura del trabajo

### 2.5.1 Metodología

Debido a la naturaleza del proyecto y a su integración directa dentro del entorno corporativo de Theia Craft, el desarrollo sigue una adaptación iterativa del Proceso Unificado, también conocido como RUP (*Rational Unified Process*) [12]. La memoria separa de forma deliberada las disciplinas de modelo de dominio, requisitos, análisis, diseño técnico, implementación y validación. Esta organización permite mantener trazabilidad entre lo que el sistema debe ofrecer, cómo se analiza conceptualmente, qué decisiones técnicas se adoptan y qué evidencia final demuestra su funcionamiento.

En la disciplina de requisitos se describe el comportamiento esperado desde el punto de vista de los actores, evitando detalles de endpoints, tablas o componentes internos. En la disciplina de análisis se estudian los casos de uso mediante la separación Vista, Controlador y Modelo. En la disciplina de diseño se concretan la arquitectura, los contratos técnicos, los servicios, los modelos de datos y las secuencias que permiten implementar esos casos. Finalmente, la disciplina de implementación y validación muestra la solución funcionando dentro de la plataforma.

Este enfoque es adecuado para un ecosistema de IA que evoluciona rápidamente, ya que permite ajustar la solución ante cambios en servicios cloud, disponibilidad de datos de facturación o soporte real de funcionalidades como RAG y prompt caching, sin mezclar los compromisos funcionales del TFG con detalles de infraestructura que pueden evolucionar.

### 2.5.2 Estructura del Trabajo

El presente Trabajo de Fin de Grado refleja el ciclo de vida del desarrollo del software y la investigación aplicada mediante las siguientes fases:

1. Investigación del problema y delimitación del alcance FinOps.
2. Modelado del dominio y especificación de requisitos.
3. Análisis conceptual de los casos de uso mediante Vista, Controlador y Modelo.
4. Diseño técnico de arquitectura, datos, interfaces y secuencias.
5. Implementación incremental del prototipo funcional.
6. Validación, revisión de resultados y conclusiones.

Esta estructura metodológica asegura no solo el cumplimiento de los objetivos académicos del TFG, sino también la viabilidad y mantenibilidad de la solución técnica en el entorno productivo real de la empresa.


# 7. Bibliografía

[1] Flexera (2024). 2024 State of the Cloud Report. Reporte anual sobre tendencias de adopción y gestión en la nube. En este informe se constata que la gestión del gasto en la nube (FinOps) ha superado a la seguridad como el principal reto corporativo por segundo año consecutivo. Disponible en: https://info.flexera.com/CM-REPORT-State-of-the-Cloud

[2] Datadog (2024). LLM Observability. Documentación técnica sobre la monitorización del rendimiento, calidad y coste de las aplicaciones basadas en Modelos de Lenguaje Grande. Disponible en: https://docs.datadoghq.com/llm_observability/ 

[3] Ternary (2024). AI Cloud FinOps. Plataforma de gestión financiera que ilustra las capacidades SaaS de terceros para la optimización de los gastos en la nube y modelos de IA. Disponible en: https://ternary.app/

[4] CloudZero (2025). The AI Cost Crisis: What AI Cost Sprawl Is And How To Fix It. Reporte sobre el crecimiento del gasto en inferencia y la necesidad de optimización autónoma. Disponible en: https://www.cloudzero.com/blog/ai-cost-crisis/

[5] Gartner (2025). Gartner Predicts 40 Percent of Enterprise Apps Will Feature Task-Specific AI Agents by 2026, Up from Less Than 5 Percent in 2025. Reporte sobre la predicción de la adopción de agentes de IA en aplicaciones empresariales. Disponible en: https://www.gartner.com/en/newsroom/press-releases/2025-08-26-gartner-predicts-40-percent-of-enterprise-apps-will-feature-task-specific-ai-agents-by-2026-up-from-less-than-5-percent-in-2025

[6] Anthropic (2024). Prompt caching with Claude. Documentación oficial sobre la reducción de latencia y la disminución de costes (hasta un 90%) al cachear tokens de contexto largos en interacciones repetitivas. Disponible en: https://www.anthropic.com/news/prompt-caching 

[7] Wei, J., Wang, X., Schuurmans, D., Bosma, M., Chi, E., Le, Q., & Zhou, D. (2022). Chain-of-Thought Prompting Elicits Reasoning in Large Language Models. Advances in Neural Information Processing Systems (NeurIPS). Documento fundacional sobre la mejora del razonamiento en LLMs mediante la generación de pasos intermedios. Disponible en: https://arxiv.org/abs/2201.11903 

[8] Yao, S., Zhao, J., Yu, D., Du, N., Narasimhan, I., Hwang, V., & Chen, K. (2023). ReAct: Synergizing Reasoning and Acting in Language Models. International Conference on Learning Representations (ICLR). Investigación original que define el paradigma mediante el cual los LLMs intercalan trazas de razonamiento con acciones en entornos externos. Disponible en: https://arxiv.org/abs/2210.03629 

[9] IBM (2025). Agentic AI vs. Generative AI. Documentación técnica sobre la autonomía, el uso de herramientas (Tool Calling) y la arquitectura ReAct en entornos corporativos. Disponible en: https://www.ibm.com/topics/agentic-ai

[10] FinOps Foundation (2024). Cost Estimation of AI Workloads. Grupo de Trabajo Oficial de FinOps para Inteligencia Artificial. Disponible en: https://www.finops.org/wg/cost-estimation-of-ai-workloads/

[11] AWS Machine Learning Blog (2024). Track, allocate, and manage your generative AI cost and usage with Amazon Bedrock. Documentación oficial de arquitectura sobre el uso de logs de invocación e Inference Profiles. Disponible en: https://aws.amazon.com/blogs/machine-learning/track-allocate-and-manage-your-generative-ai-cost-and-usage-with-amazon-bedrock/

[12] Larman, C. (2004). *Applying UML and Patterns: An Introduction to Object-Oriented Analysis and Design and Iterative Development* (3.ª ed.). Prentice Hall PTR.

[13] FastAPI. Documentación oficial. Disponible en: https://fastapi.tiangolo.com

[14] SQLAlchemy. Documentación oficial de SQLAlchemy 2.0 y asyncio. Disponible en: https://docs.sqlalchemy.org/en/20/

[15] TanStack Query. Documentación oficial. Disponible en: https://tanstack.com/query/latest

[16] Martin, R. C. (2017). *Clean Architecture: A Craftsman's Guide to Software Structure and Design*. Prentice Hall PTR.

[17] AWS. *Analyzing your costs and usage with AWS Cost Explorer*. Disponible en: https://docs.aws.amazon.com/cost-management/latest/userguide/ce-what-is.html

[18] AWS. *Activating user-defined cost allocation tags*. Disponible en: https://docs.aws.amazon.com/awsaccountbilling/latest/aboutv2/activating-tags.html

[19] AWS. *GetCostAndUsage - AWS Billing and Cost Management API Reference*. Disponible en: https://docs.aws.amazon.com/aws-cost-management/latest/APIReference/API_GetCostAndUsage.html

[20] AWS. *Application inference profiles - Amazon Bedrock*. Disponible en: https://docs.aws.amazon.com/bedrock/latest/userguide/cost-mgmt-application-inference-profiles.html

[21] AWS. *IAM principal attribution - Amazon Bedrock*. Disponible en: https://docs.aws.amazon.com/bedrock/latest/userguide/cost-mgmt-iam-principal-tracking.html

[22] AWS. *GetAnomalies - AWS Billing and Cost Management API Reference*. Disponible en: https://docs.aws.amazon.com/aws-cost-management/latest/APIReference/API_GetAnomalies.html
