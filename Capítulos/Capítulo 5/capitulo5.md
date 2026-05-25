# 6. Conclusiones y futuras líneas de actuación

## 6.1 Cumplimiento de objetivos

El objetivo general del TFG era diseñar, desarrollar y validar un módulo de gestión financiera en la nube, orientado a la Misión FinOps, para integrarlo en una plataforma de gobierno de IA como parte de un CAIO Virtual. La solución implementada cumple este objetivo dentro del alcance de un prototipo funcional: incorpora un recorrido completo para consultar costes de IA, atribuir gasto a agentes cuando la cuenta cloud lo permite, detectar anomalías y presentar primeras acciones de optimización sobre agentes Bedrock.

La implementación no se plantea como un producto cerrado, sino como un prototipo funcional verificable dentro de un sistema real. Esto encaja con la naturaleza del proyecto: partir de un escenario empresarial existente, aplicar una metodología de ingeniería del software y construir una solución que conecte requisitos, análisis, diseño e implementación.

| Objetivo específico | Evidencia de cumplimiento | Resultado obtenido |
| --- | --- | --- |
| OE1. Definir requisitos y modelo de dominio. | Capítulo 3: actores, casos de uso, modelo de dominio, glosario y requisitos suplementarios. | Se acotó el problema FinOps y se separaron las misiones CU-07 a CU-12. |
| OE2. Definir análisis y diseño arquitectónico. | Capítulo 4: arquitectura, clases, paquetes, datos, despliegue y secuencias. | Se conectó el dominio FinOps con FastAPI, Next.js, PostgreSQL, AWS y servicios internos. |
| OE3. Desarrollar un prototipo funcional de la Misión FinOps. | Capítulo 5: navegación, pantallas, endpoints y servicios implementados. | Se obtuvo una solución navegable para costes, anomalías y optimización de agentes. |
| OE4. Evaluar el prototipo en entorno controlado. | Capítulos 5 y 6: datos reales cuando están disponibles, escenarios de validación controlados y contratos Swagger. | Se puede evaluar el flujo completo aunque AWS no tenga aún toda la facturación o atribución preparada. |

### 6.1.1 OE1: requisitos y modelo de dominio

El primer objetivo se considera cumplido mediante la identificación de entidades y relaciones propias del dominio FinOps: organización, agente de IA, proveedor cloud, credencial, misión, coste, coste atribuido, coste no asociado, anomalía y uso LLM. Esta base permitió evitar una visión genérica de facturación y centrar el trabajo en el gobierno financiero de sistemas de IA.

También se definieron casos de uso con distintos niveles de prioridad. Los casos CU-07 a CU-12 constituyen el alcance principal de esta entrega porque cubren visibilidad, atribución, alerta y optimización inicial. La separación entre coste total y coste por agente fue especialmente importante: el coste total puede consultarse sin preparar etiquetas de atribución, mientras que el coste por agente requiere una configuración adicional en AWS.

### 6.1.2 OE2: análisis, diseño y arquitectura

El segundo objetivo se considera cumplido al transformar los requisitos en una arquitectura concreta. La solución mantiene la separación entre frontend, API, servicios de dominio, repositorios y modelos de persistencia. Esta organización permite que cada caso de uso tenga un camino claro desde la interfaz hasta el dato:

```text
Pantalla -> API FastAPI -> Servicio de dominio -> Repositorio -> Tabla o API externa
```

El diseño también diferencia entre tres fuentes de información:

- AWS Cost Explorer, para costes Bedrock del cliente.
- `llm_usage_logs`, para coste estimado del uso LLM de la propia plataforma.
- Amazon Bedrock Agents, para optimización RAG y prompt caching.

Esta separación reduce ambigüedad. No se mezclan facturas cloud, estimaciones internas y cambios de configuración de agentes en una sola abstracción.

### 6.1.3 OE3: prototipo funcional FinOps implementado

El tercer objetivo se considera cumplido mediante una implementación navegable dentro de la plataforma. El usuario puede entrar en Misiones, activar el bloque FinOps, ejecutar o revisar cada misión y comprobar los seis casos de uso desde la página Costs, que actúa como vista principal de la entrega.

La solución implementada cubre:

- CU-07: coste total AWS Bedrock.
- CU-08: coste por agente y coste no asociado.
- CU-09: anomalías de gasto con detector local y auditoría.
- CU-10: coste LLM estimado de la plataforma.
- CU-11: simulación y prueba controlada de recuperación RAG.
- CU-12: dry-run y aplicación controlada de prompt caching.

El valor del prototipo funcional está en que no se limita a una pantalla informativa. Incorpora persistencia, endpoints, auditoría y mecanismos de revisión que permiten explicar el flujo completo desde la interfaz hasta los servicios backend.

### 6.1.4 OE4: validación controlada

El cuarto objetivo se considera cumplido con una estrategia de validación controlada. El flujo principal de la aplicación está planteado para trabajar con datos reales cuando las credenciales y permisos AWS están preparados, y además se han previsto escenarios reproducibles para poder evaluar el recorrido completo en local.

Esta decisión es adecuada para un TFG porque evita que la evaluación dependa de factores externos como la disponibilidad de gasto reciente en AWS, la propagación de etiquetas de coste o la existencia de agentes Bedrock con Knowledge Bases reales. La validación controlada queda acotada como apoyo metodológico; no sustituye al flujo real descrito en la solución.

### 6.1.5 Protocolo de validación del alcance implementado

Para que la validación no quede reducida a una afirmación general, se separa por caso de uso qué se comprueba, qué datos intervienen y qué evidencia permite revisarlo. Cuando AWS proporciona datos reales, el sistema los utiliza; cuando no existe histórico suficiente, señal de atribución o agente compatible, se emplean datos de validación deterministas para comprobar el recorrido funcional sin inventar capacidades del proveedor.

| Caso | Datos usados | Resultado esperado | Resultado obtenido | Evidencia |
| --- | --- | --- | --- | --- |
| CU-07 Coste total AWS Bedrock | Registros de Cost Explorer disponibles o datos deterministas de facturación. | Mostrar el coste total del periodo aunque no exista atribución por agente. | La vista `Costs` y el contrato de billing presentan total, moneda y periodo. | Figuras 5.2 y 5.6; endpoints `GET /api/v1/billing/total` y `GET /api/v1/billing/costs`. |
| CU-08 Coste por agente | Señal `theia-agent-id` cuando está visible en Cost Explorer o registros de validación equivalentes. | Separar coste atribuido por agente y coste no asociado. | El sistema conserva `unmatched_cost` y no inventa asociaciones cuando falta la señal. | Figura 5.6; listado 5.1. |
| CU-09 Anomalías | Línea base de 14 días y día observado. | Detectar desviaciones relevantes y dejar evidencia revisable. | El detector local calcula severidad, umbral y resultado, y la ejecución administrativa genera auditoría. | Figura 5.6; listados 5.2 y 5.5. |
| CU-10 Coste LLM estimado | Registros internos de llamadas LLM con tokens y precio estimado. | Mostrar coste estimado por fuente, proveedor y modelo sin presentarlo como factura oficial. | La API marca el resultado como estimado y agrega llamadas, tokens y coste. | Figura 5.8; endpoint `GET /api/v1/llm/costs`. |
| CU-11 Optimización RAG | Agentes Bedrock descubiertos, Knowledge Bases disponibles o metadatos sincronizados. | Simular perfiles de recuperación sin persistir cambios en AWS y permitir prueba controlada si procede. | La pantalla muestra agente, estado RAG, configuración de recuperación y resultado de simulación; la invocación de prueba queda reservada a administrador. | Figura 5.10; endpoints `GET /api/v1/agents/{agent_id}/rag-optimization`, `POST /api/v1/agents/{agent_id}/rag-optimization/simulate` y listado 5.5. |
| CU-12 Prompt caching | Estado del agente y configuración avanzada compatible cuando existe. | Ejecutar dry-run, exigir confirmación administrativa y aplicar cambios solo si hay soporte real. | La solución informa elegibilidad; si falta `promptOverrideConfiguration.promptConfigurations`, conserva la simulación y bloquea la mutación. | Figura 5.12; endpoints `GET /api/v1/agents/{agent_id}/prompt-cache`, `POST /api/v1/agents/{agent_id}/prompt-cache/simulate` y listado 5.5. |

Como comprobación técnica complementaria, se ejecutó una batería enfocada de pruebas sobre servicios y APIs FinOps: `test_billing_service.py`, `test_billing_api.py`, `test_llm_costs_api.py` y `test_agent_optimization_api.py`. El resultado fue de 32 pruebas superadas. Esta evidencia no sustituye la validación funcional con AWS, pero refuerza que las reglas principales de agregación, anomalías, coste LLM estimado y optimización de agentes están cubiertas a nivel de servicio y contrato.

## 6.2 Discusión de resultados y decisiones

### 6.2.1 Separar coste total y coste por agente

Una de las decisiones más importantes fue dividir el coste AWS en dos casos de uso: CU-07 para coste total y CU-08 para coste por agente. Inicialmente podría parecer más simple mostrar una única vista de costes, pero esa simplificación habría ocultado una realidad técnica: AWS puede exponer coste de Bedrock antes de que exista una señal fiable para atribuirlo a un agente concreto.

Separar ambos casos de uso mejora la explicación y la utilidad del sistema. CU-07 permite responder rápido cuánto se está gastando. CU-08 añade una capa de madurez: saber quién o qué agente concentra ese gasto cuando existe una señal de atribución fiable. Esta separación también evita bloquear el avance del usuario si la cuenta AWS todavía no tiene activada `theia-agent-id` como `cost allocation tag` o si esa señal aún no aparece en Cost Explorer.

### 6.2.2 Mantener el coste no asociado

El uso de `unmatched_cost` es una decisión de transparencia. Cuando existe coste Bedrock sin agente asociado, el sistema lo muestra aparte en vez de ignorarlo. Desde el punto de vista de gobierno, esto es más útil que presentar solo el coste atribuido, porque permite detectar que todavía falta preparación de etiquetas, perfiles, roles o propagación de datos en AWS.

Esta decisión aporta valor incluso cuando no se puede completar el análisis por agente: el usuario sigue teniendo visibilidad del gasto y sabe qué parte necesita trabajo posterior de atribución.

### 6.2.3 Validación reproducible

Los escenarios de validación reproducibles no sustituyen la integración real, sino que aseguran que el recorrido de evaluación pueda repetirse de forma consistente. En un proyecto dependiente de servicios cloud, hay variables externas que no siempre se pueden controlar: retrasos de Cost Explorer, falta de gasto histórico, ausencia de Knowledge Bases o limitaciones temporales de una cuenta de prueba.

Por eso se han definido datos de validación deterministas para billing, uso LLM y agentes de optimización. La palabra determinista es importante: cada ejecución prepara un escenario conocido, explicable y alineado con los casos de uso. Esto permite demostrar la solución sin improvisar datos ni depender del estado exacto de AWS en el momento de la evaluación.

### 6.2.4 Estimación frente a factura oficial

CU-10 diferencia entre coste estimado y factura oficial. Esta distinción es necesaria porque el coste LLM de la plataforma se calcula a partir de registros internos de tokens y precio almacenado, no desde la factura final de un proveedor.

El resultado no pretende sustituir a OpenAI, Anthropic, Azure OpenAI, OpenRouter o AWS Billing. Su valor está en ofrecer observabilidad operativa dentro de la plataforma: qué funcionalidades consumen más tokens, qué modelos se usan y cuánto coste estimado genera el propio CAIO Virtual.

### 6.2.5 Optimización condicionada

CU-11 y CU-12 se han diseñado como optimizaciones condicionadas. Reducir chunks RAG puede reducir contexto, pero también puede afectar cobertura. Activar prompt caching puede reducir coste en llamadas repetidas, pero depende del modelo, región, TTL, prefijo cacheable y patrón real de tráfico.

Por eso ambas misiones presentan simulación antes de acción. CU-11 permite probar una decisión de recuperación sin persistir cambios en el agente. CU-12 exige dry-run y confirmación administrativa antes de aplicar una modificación real. Esta forma de diseño evita prometer ahorro automático y mantiene el control en manos del administrador.

## 6.3 Limitaciones asumidas

El alcance desarrollado es suficiente para evaluar el módulo, pero deja limitaciones claras para trabajo posterior.

| Limitación | Impacto | Tratamiento actual |
| --- | --- | --- |
| Cost Explorer publica datos con retraso. | Puede no haber coste reciente visible aunque exista uso. | Se explica la semántica temporal y se permite usar un escenario controlado. |
| La atribución por agente depende de una señal visible en Cost Explorer, como `cost allocation tags`, perfiles, identidad o mecanismo de invocación. | CU-08 puede mostrar coste no asociado si `theia-agent-id` no está preparada, no es retroactiva o aún no se refleja en la facturación. | Se conserva `unmatched_cost` para no ocultar gasto ni asumir atribución no verificada. |
| CU-09 necesita histórico diario. | Sin 14 días previos no hay línea base suficiente. | El escenario de validación prepara una línea base y un día observado. |
| CU-10 es estimado. | No sustituye una factura oficial. | La API marca `estimated = true`. |
| CU-11 depende de Knowledge Bases. | No todos los agentes Bedrock son candidatos. | La interfaz indica aplicabilidad y permite simulación controlada. |
| CU-12 depende de modelo, región y configuración avanzada. | No todos los agentes pueden activar prompt caching ni todos exponen `promptOverrideConfiguration.promptConfigurations`. | Se usa elegibilidad, dry-run y confirmación; si falta soporte compatible, no se aplica mutación real. |

Estas limitaciones no invalidan el trabajo; delimitan correctamente la primera versión. Además, muestran una ventaja de haber seguido un proceso metodológico: se sabe qué parte está implementada, de qué depende cada caso de uso y qué condiciones habría que preparar para evolucionarlo.

## 6.4 Recomendaciones

### 6.4.1 Preparar una cuenta AWS real para atribución completa

La recomendación principal es preparar una cuenta AWS con uso real de Bedrock y una estrategia de atribución estable. Para CU-08, lo más importante es activar una clave como `theia-agent-id` como `cost allocation tag`, asociarla a la identidad, sesión, perfil, proyecto o mecanismo de invocación que genere el consumo, y generar uso nuevo para que Cost Explorer pueda exponerlo.

Esto permitiría pasar de una validación controlada a una validación prolongada con datos reales de coste por agente.

### 6.4.2 Reforzar pruebas automáticas

El módulo ya cuenta con servicios y endpoints diferenciados, lo que facilita ampliar pruebas unitarias e integración. La recomendación es reforzar especialmente:

- agregaciones de coste total;
- cálculo de `unmatched_cost`;
- detección de anomalías con distintos patrones diarios;
- coste LLM estimado por fuente y modelo;
- dry-run y aplicación de prompt caching.

Esto encaja con la idea de auditoría diseño-código aplicada en la memoria: no basta con que el sistema funcione en una ejecución controlada; conviene verificar que la implementación sigue correspondiendo al diseño.

### 6.4.3 Mejorar la observabilidad de misiones

Otra recomendación es mejorar la visibilidad del estado interno de cada misión. La pantalla actual guía al usuario, pero en futuras versiones sería útil tener más información de diagnóstico: última sincronización, permisos detectados, datos disponibles por periodo, estado de tags y auditorías relacionadas.

Esto ayudaría tanto al usuario final como al equipo técnico en una puesta en marcha real.

### 6.4.4 Mantener documentación técnica del módulo

Además de la memoria, conviene mantener documentación técnica específica para despliegue, configuración y operación del módulo: variables de entorno, permisos AWS necesarios, pasos de sincronización, datos mínimos para validación y recorrido funcional de los seis casos de uso. Esta documentación no forma parte del análisis académico principal, pero facilita la continuidad del desarrollo y reduce la dependencia del conocimiento tácito del equipo.

## 6.5 Futuras líneas de actuación

### 6.5.1 Alertas proactivas

CU-09 detecta anomalías, pero una evolución natural es convertir esa detección en alertas proactivas. El sistema podría notificar al administrador por email, Slack o un panel de notificaciones cuando el detector encuentre un pico crítico.

La arquitectura ya dispone de `audit_logs`, tareas periódicas y servicios de notificación, por lo que esta línea puede crecer sin rediseñar todo el módulo.

### 6.5.2 Informes ejecutivos exportables

Una segunda línea de evolución es generar informes PDF o HTML para dirección. Estos informes podrían resumir coste total, coste por agente, gasto no asociado, anomalías y recomendaciones de optimización.

El valor de esta línea es convertir la información de la página Costs en un artefacto compartible para reuniones de seguimiento o auditorías internas.

### 6.5.3 Recomendaciones FinOps automáticas

El módulo actual muestra datos y permite acciones concretas. Una evolución más ambiciosa sería generar recomendaciones FinOps automáticas: investigar agentes con mayor coste, sugerir etiquetas faltantes, proponer límites de presupuesto o recomendar perfiles RAG más conservadores cuando una consulta sea simple.

Esta línea encaja especialmente con el concepto de CAIO Virtual, porque transforma observabilidad en asistencia activa.

### 6.5.4 Integración más profunda con AWS

Otra línea de trabajo es ampliar la integración con AWS Bedrock y AWS Cost Anomaly Detection. Actualmente AWS Cost Anomaly Detection se usa como señal opcional de lectura. En una versión posterior, la plataforma podría gestionar monitores, presupuestos o suscripciones de alerta si la organización concede permisos adecuados.

También sería posible ampliar el soporte a perfiles de inferencia de aplicación, tags por principal IAM y análisis más fino de modelos fundacionales usados.

### 6.5.5 Ampliación multi-cloud

Aunque el TFG se centra en AWS para las misiones FinOps principales, la arquitectura ya contempla conectores con otros proveedores. Una línea futura es trasladar el mismo patrón a GCP y Azure:

```text
coste total -> coste atribuible -> anomalía -> coste interno -> optimización
```

Esta evolución debería mantener la misma disciplina metodológica: primero requisitos y dominio, después análisis y diseño, y finalmente implementación trazable por casos de uso.

## 6.6 Cierre

El TFG demuestra que es posible integrar un módulo FinOps en un CAIO Virtual sin tratar la facturación como una tabla aislada. La solución desarrollada conecta costes cloud, agentes de IA, uso LLM, auditoría y optimización en un recorrido único para el usuario.

La aportación principal no está solo en consultar datos de AWS, sino en convertirlos en una experiencia de gobierno: misiones guiadas, evidencia visible en pantallas, datos persistidos, endpoints verificables y decisiones de diseño que reconocen las condiciones reales de los servicios cloud.

Como base funcional, el módulo queda preparado para crecer. La separación entre casos de uso, servicios y fuentes de datos permite ampliar la solución sin perder trazabilidad. Esa trazabilidad es precisamente la clave del trabajo: lo implementado puede explicarse desde los objetivos iniciales, los requisitos, el diseño y el recorrido final que ve el usuario.
