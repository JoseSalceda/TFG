# 1 Introducción

## 1.1 Contexto del proyecto

Este TFG busca diseñar e implementar una arquitectura *middleware* para la gestión eficiente y optimización de costes en entornos corporativos. Este proyecto se hará en conjunto y por necesidad de Theia Craft, empresa colaboradora en la que se desarrolla este TFG. En Theia Craft se busca crear un agente virtual que pueda actuar como trabajador de una empresa, gestionando sus tareas diarias, automatizando procesos y optimizando la productividad. Son agentes de IA que se integran en el ecosistema de la organización y que aportan un gran valor.

## 1.2 Problema identificado

Muchas organizaciones, incluso las más grandes, siguen sin tener claro cómo adoptar una IA que avanza tan rápido que cuesta mantenerse al día con sus avances. Estas empresas saben de la utilidad de la IA y cómo usarla de forma correcta puede hacerla una herramienta valiosísima para su negocio. Uno de los muchos desafíos, y que constituye el núcleo central de este proyecto, es la gestión de costes. El objetivo principal de las organizaciones es maximizar su rentabilidad, por lo que optimizar cada euro es clave. El problema es que el dinero de las empresas se esfuma por cualquier lugar y saber entender dónde se gasta, cómo se gasta, si es un coste que vale la pena y mucho más no es fácil [1]. 

En el ámbito de la IA y la nube, los costes asociados a las peticiones de LLMs y el despliegue de agentes (infraestructura, bases de datos vectoriales y *tokens*) suelen comportarse como una caja negra, generando gastos imprevistos. Las principales plataformas cloud (AWS, GCP, Azure) ofrecen herramientas para gestionar los costes, pero suelen carecer de una visibilidad granular casi en tiempo real a nivel de interacciones y *tokens* consumidos (el nivel micro de los LLMs). En este contexto, la disciplina corporativa de FinOps aplicada a la Inteligencia Artificial (a menudo denominada *GenOps* o *LLMOps Financial Management*) se convierte en una necesidad inmediata, ya que la rentabilidad y el control del gasto son pilares fundamentales para la existencia y sostenibilidad de cualquier empresa.

## 1.3 Solución propuesta

Para dar respuesta a esta problemática, el presente TFG propone el diseño y desarrollo de varias "Misiones FinOps" integradas dentro del ecosistema de agentes virtuales de Theia Craft para las empresas que utilicen la plataforma. En lugar de depender de paneles de control estáticos (*dashboards*) que requieren la revisión manual de un ingeniero, la solución consiste en dotar a un Agente de IA de herramientas (*Action Groups*) que le permitan interactuar directamente con las APIs de facturación y monitorización de la nube (específicamente Amazon Web Services - AWS para este TFG).

Las misiones FinOps que se propondrán a lo largo de este TFG serán variadas. Las más generales serán de visibilidad de costes, ya que sin visibilidad del saldo y gasto es imposible tomar decisiones estratégicas informadas. Las misiones específicas serán de optimización de costes, buscando, una vez se tenga una visión clara de dónde se gasta el dinero, darle la visibilidad necesaria al negocio para que pueda redirigir sus gastos de manera correcta.

# 2 Marco teórico

## 2.1 Estado del arte

### 2.1.1 Contexto Theia Craft

Theia Craft es una consultoría informática especializada en B2B, a diferencia del modelo B2C tradicional. Es una *startup* con apenas un año de vida fundada por Kiyoshi Omaza, ingeniero con mucha experiencia en grandes empresas tecnológicas de Madrid y tutor en este TFG. Actualmente Theia Craft está centrada en el desarrollo de tecnologías para ayudar a empresas a mejorar la toma de decisiones a través de una propuesta que combina investigación del funcionamiento de la normativa, de la empresa en la que se implementa y que está potenciada por el uso de varias tecnologías. Entre ellas destaca la IA mediante el uso de trabajadores digitales que disparan la productividad de los empleados, ya que así se pueden centrar en las tareas que verdaderamente importan.

Una de las peticiones más repetidas por los distintos clientes interesados por la propuesta de Theia Craft es la de aportar mayor visibilidad y control sobre las finanzas de la empresa, ya que con el paso de los años se han visto obligados a usar varias herramientas cloud a la vez. Algunas empresas han comentado que usan hasta cinco proveedores cloud distintos, cada uno con sus propias herramientas y formas de gestionar los costes. Esta fragmentación, sumada a la adopción acelerada de asistentes de IA y flujos agénticos, ha provocado que el coste de la inferencia (las llamadas a LLMs) se vuelva impredecible. La falta de herramientas unificadas que crucen el nivel táctico (consumo de *tokens* y contexto) con el nivel estratégico (facturación y retorno de inversión) hizo que Theia Craft viera la necesidad de investigar y desarrollar una solución FinOps nativa e integrada en su ecosistema.


### 2.1.2 Soluciones existentes

En la actualidad, el reto de la gestión de costes de IA se está abordando desde tres ángulos principales, cada uno con fortalezas y limitaciones para un entorno de IA Agéntica:

1. Herramientas nativas Cloud (Cloud Cost Management): Plataformas como AWS Cost Explorer, GCP Cloud Billing y Azure Cost Management han comenzado a incorporar funcionalidades específicas para recursos de *machine learning* [1]. Proporcionan un excelente control a nivel macroestructural (coste de instancias, bases de datos vectoriales), pero por defecto carecen de la granularidad requerida para desglosar el coste exacto a nivel de interacciones individuales, misiones de un agente o consumo de *tokens*, requiriendo desarrollos adicionales e interpretación manual.

2. Plataformas de Observabilidad de LLMs (LLMOps): Herramientas de terceros como LangSmith, Helicone o Datadog LLM Observability se centran en el nivel micro. Capturan cada petición, la longitud del *prompt*, el rendimiento y los *tokens* consumidos, calculando un coste estimado [2]. Sin embargo, a menudo su facturación es estimada y no cruza directamente con la factura real de la nube del cliente, separando la telemetría operativa de la realidad financiera corporativa.

3. Plataformas FinOps especializadas: Soluciones SaaS de gestión en la nube (ej. Finout, Ternary, CloudZero) están evolucionando para incluir *unit economics* de IA [3] [4]. Permiten asignar costes complejos uniendo logs y facturas. El inconveniente radica en que son herramientas pasivas o de reporte (*dashboards*), pensadas para ser consumidas por un equipo financiero o un ingeniero. Exigen la exportación de datos a plataformas externas y no aprovechan la proactividad del agente de IA para actuar sobre esos datos.

El vacío en el estado del arte actual reside en la IA Agéntica aplicada a FinOps. Mientras las soluciones existentes se enfocan en otorgar visibilidad para que un humano tome las decisiones, la propuesta de este TFG busca integrar la observabilidad financiera directamente en las capacidades (*Action Groups*) del Agente Virtual comercializado por Theia Craft, permitiendo una orquestación y reporte de costes autónomo.

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

Los costes se gestionan a nivel macro mediante AWS Cost Explorer, pero para obtener visibilidad micro (*tokens*), es necesario configurar CloudWatch Logs, enviar los registros de invocación a un grupo específico, y procesarlos externamente, un flujo complejo que carece de inmediatez y centralización por defecto.

De las tres opciones evaluadas, AWS destaca como la alternativa principal. Se percibe más desarrollada y madura que las otras dos opciones.

### 2.1.7.2 GCP

Google Cloud Platform ofrece Vertex AI para modelos generativos y Dialogflow CX para agentes conversacionales robustos. La facturación se consolida en Cloud Billing y se puede exportar a BigQuery para un análisis detallado (Standard usage cost export).

Aunque GCP permite un análisis granular cruzando BigQuery con los logs de operaciones de los agentes, requiere conocimiento avanzado de SQL y una configuración manual de permisos y exportaciones para correlacionar los costes de infraestructura con el uso específico de los LLMs.

Es una plataforma bastante robusta y con gran número de opciones, pero se percibe que todavía está algo lejos de su potencial definitivo.

### 2.1.7.3 Azure

Microsoft Azure provee Azure OpenAI Service y Azure Machine Learning. Está profundamente integrado con su ecosistema enterprise, facturando por *tokens* procesados en sus *endpoints* online. Azure Cost Management ofrece herramientas para el etiquetado y presupuestos, pero al igual que sus competidores, cruzar el uso técnico diario con el gasto financiero corporativo (FinOps) de múltiples modelos y endpoints en diferentes suscripciones requiere soluciones de terceros o desarrollos a medida.

Es la más hermética de las tres opciones y además los requisitos de acceso a ella son más estrictos que los de AWS y GCP.

### 2.1.8 Plataforma que usaré

Las tres plataformas cloud mencionadas son las más grandes de la industria, cada una con sus ventajas y desventajas. De las tres, actualmente la actividad en Theia Craft se centra en dos, AWS y GCP. En este TFG, AWS será la plataforma que se utilizará para desplegar la solución propuesta, ya que, tras varios meses de uso de ambas, AWS ha demostrado ser más responsiva y ofrecer un mayor número de herramientas para el control de costes cloud. Obviamente, es posible que con el paso del tiempo o gracias a alguna opción concreta GCP sea realmente mejor, pero finalmente se usará AWS.

### 2.1.9 Tabla 1. Comparativa de características FinOps.

| Característica FinOps (GenOps) | AWS | GCP | Azure |
| :--- | :--- | :--- | :--- |
| **Visibilidad de Costes (Macro)** | Sí | Sí | Sí |
| **Exportación a Motor Analítico SQL** | A medias | Nativo | A medias |
| **Visibilidad de Nivel Micro (*Tokens*)** | Sí | Sí | Sí |
| **Detección de Anomalías de IA** | Sí | Sí | Sí |
| **Atribución de Coste por Agente** | Alta | Media | Media |
| **Acceso API (*Tool Calling*)** | Sí | Sí | Sí |


## 2.2 Justificación de la propuesta

La gestión financiera en la nube (Cloud Financial Management) se ha basado históricamente en métricas predecibles, como el tiempo de actividad de un servidor o el almacenamiento consumido. Sin embargo, los Modelos de Lenguaje Grande (LLMs) introducen un paradigma de facturación dinámico basado en el consumo de *tokens*. Este modelo de precios fluctúa enormemente dependiendo de la longitud del contexto y de los bucles de razonamiento interno que realiza la IA, provocando que el rastreo manual de estas micro-transacciones a través de miles de interacciones corporativas sea inasumible para los equipos humanos [10].

Ante este cambio, el desarrollo de un *middleware* adaptado aborda el problema de raíz. Implementar misiones FinOps dedicadas a aportar visibilidad a la empresa en todos los ámbitos, específicamente en el financiero, da un valor sustancial al ecosistema de Theia Craft, ya que automatiza la extracción, el cruce y la estructuración de datos financieros complejos. En lugar de depender de la revisión periódica de paneles de control por parte de un ingeniero, la arquitectura propuesta permite transformar logs técnicos anidados (JSON) en una única plataforma agéntica, agilizando drásticamente la detección de sobrecostes y la toma de decisiones [4].

Es fundamental dejar claro que este reto no puede resolverse utilizando Inteligencia Artificial generativa estándar. Un LLM aislado carece de acceso a la infraestructura en tiempo real y no está diseñado para realizar cálculos matemáticos deterministas con fiabilidad. Por este motivo, el proyecto exige el despliegue de una arquitectura de IA Agéntica. El sistema requiere capacidad para decidir qué herramientas usar (*Tool Calling*), interactuando de forma autónoma con los servicios de AWS a través de *Action Groups* (funciones Lambda en AWS) para garantizar que los datos expuestos reflejen la realidad absoluta de la facturación y no alucinaciones del modelo.

El núcleo que aporta viabilidad práctica a este Trabajo de Fin de Grado es, precisamente, su profunda integración técnica. La capacidad del agente para conectarse a las APIs de AWS Cost Explorer y Amazon CloudWatch es lo que convierte una mera prueba de concepto conversacional en una herramienta de gobierno financiero real. Si el agente no pudiera auditar el identificador exacto de una sesión (*sessionId*) o los *tokens* exactos consumidos en un flujo productivo, la solución carecería de impacto en los procesos empresariales.

Esta integración responde además a una necesidad estricta de trazabilidad. Las auditorías FinOps requieren correlacionar el impacto macroeconómico (la factura mensual) con su origen microscópico (qué agente o *prompt* generó el gasto). Gestionar este nivel de contexto entre distintos servicios de monitorización justifica plenamente el uso de un orquestador inteligente en lugar de un simple *script* de extracción de datos.
 
Por último, la selección de Amazon Bedrock como infraestructura base responde a los requisitos de seguridad y al contexto tecnológico de Theia Craft. Aunque el mercado ofrece frameworks de orquestación alternativos de código abierto (como LangChain o LangGraph) que podrían alojarse en servidores propios, Bedrock proporciona capacidades nativas como el *Model Invocation Logging* y los perfiles de inferencia cruzados con AWS IAM. Esto permite que la arquitectura herede los estándares de seguridad corporativa (*Zero Trust*), garantizando que el agente accede a los datos de facturación bajo el principio de mínimo privilegio [11]. Esta suma de capacidades nativas e integración segura hace de AWS el entorno tecnológico idóneo para el desarrollo de la propuesta.


## 2.3 Solución propuesta

Este primer capítulo se centra en dejar claro el contexto técnico de la propuesta. Es posible que en el futuro, al igual que ya ha sucedido desde las anteriores entregas, haya cambios en la arquitectura, solución o ámbito.

La solución propuesta actual es un "*Pipeline* de FinOps & Gobernanza" embebido en el producto de Theia Craft. Técnicamente constará de:

- Un modelo de datos unificado para representar "Misiones" e "Introspección de Sistemas", agnosticizando las complicaciones técnicas ligadas a la nube específica (AWS).
- Un flujo de usuario (UI) interactivo híbrido, donde el sistema realiza acciones en la nube (auto-detectar, auto-habilitar) cuando tiene permisos suficientes (*workerCanDoIt*), pero ofrece instrucciones granulares e inserciones de código "en vivo" cuando la configuración requiere seguridad o derechos exclusivos del cliente humano administrador.
- Conexión de arquitecturas sin servidor (*Serverless*), como AWS Lambda, orquestadas y desplegadas bajo la instrucción del middleware para actuar como recolectores locales de datos de costes, superando las limitaciones nativas de exportación de ciertas sub-herramientas cloud.


## 2.4 Objetivos

Para dar respuesta a la problemática planteada y estructurar el desarrollo de la solución, se han definido un objetivo general y una serie de objetivos específicos que guiarán las distintas fases del proyecto.

### 2.4.1 Objetivo general

Diseñar, desarrollar y validar un módulo de gestión financiera en la nube (Misión FinOps) basado en Inteligencia Artificial Agéntica para su integración en la plataforma corporativa Theia Officer de Theia Craft. Esta arquitectura middleware, desplegada sobre los servicios de Amazon Web Services (AWS), tiene como fin principal proporcionar a las organizaciones visibilidad financiera granular, prevenir sobrecostes y automatizar la optimización del gasto derivado del uso de Modelos de Lenguaje Grande (LLMs).

### 2.4.2 Objetivos específicos

Para alcanzar el objetivo general propuesto, el proyecto se desglosa en los siguientes objetivos específicos, los cuales se encuentran directamente mapeados con las fases de ingeniería del software y los capítulos de este documento:

1. Definir la disciplina de requisitos y elaborar el modelo de dominio del sistema.
2. Definir la disciplina de análisis y diseño arquitectónico que conduzca a cumplir con el objetivo general, definiendo la infraestructura en la nube (AWS), las integraciones de red y la orquestación de los agentes de IA necesarios para que el flujo funcione de manera óptima.
3. Desarrollar una primera iteración en forma de Producto Mínimo Viable (MVP) de la "Misión FinOps", que responda fielmente a los requisitos, análisis y diseño realizados en las fases anteriores.
4. Evaluar el prototipo, validando la exactitud de la extracción de costes en un entorno controlado y analizando el valor aportado a la toma de decisiones empresariales.


## 2.5 Estructura del trabajo

### 2.5.1 Metodología

Debido a la naturaleza del proyecto y a su integración directa dentro del entorno corporativo de Theia Craft, el desarrollo se rige por una metodología ágil, tomando como referencia el marco de trabajo Scrum adaptado a un contexto de investigación y desarrollo (I+D).

Este enfoque iterativo e incremental es fundamental para un ecosistema de IA que evoluciona rápidamente, permitiendo adaptar la arquitectura ante posibles actualizaciones en las APIs de los proveedores Cloud o en el comportamiento de los LLMs.

### 2.5.2 Estructura del Trabajo

El presente Trabajo de Fin de Grado busca que se refleje el ciclo de vida del desarrollo del software y la investigación aplicada, que, mediante la metodología Scrum, se articula en las siguientes fases:

1. Investigación y Toma de Requisitos
2. Diseño y Modelado
3. Implementación Iterativa (Sprints)
4. Validación y Pruebas Continuas
5. Revisión y Despliegue

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