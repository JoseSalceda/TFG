# 5. Disciplina de Análisis

## 5.1 Introducción

La disciplina de análisis toma como punto de partida el modelo de dominio y los casos de uso definidos en el capítulo 2. Su objetivo no es repetir qué necesita el usuario, sino refinar esos requisitos con un lenguaje más cercano al desarrollo: arquitectura, colaboraciones entre componentes, clases conceptuales y organización lógica del módulo.

En este TFG el sistema no se desarrolla desde cero. Theia Officer ya existe como plataforma de gobierno de agentes de IA y este trabajo se centra en una parte concreta: el módulo FinOps para visibilidad y control financiero de agentes de IA en AWS. Por tanto, el análisis no redefine la aplicación completa, sino que estudia cómo encajan las capacidades financieras dentro de la arquitectura existente.

El capítulo 2 prioriza los nuevos casos de uso FinOps mediante MoSCoW. CU-07 y CU-08 aparecen como Must, CU-09 a CU-11 como Should y CU-12 a CU-13 como Could. En este capítulo se analizan todos ellos, pero con distinto nivel de profundidad: CU-07 se detalla por ser la base de datos, pantallas y agregaciones sobre las que se apoyan los demás; CU-08 se diseña de forma condicionada porque requiere histórico real; CU-09 a CU-13 se tratan como evolución del módulo.

## 5.2 Análisis de la arquitectura

### 5.2.1 Arquitectura existente de Theia Officer

Theia Officer utiliza una arquitectura por capas: frontend Next.js, backend FastAPI, servicios de dominio, repositorios de acceso a datos y modelos SQLAlchemy. La documentación técnica del proyecto describe el flujo general como `Frontend -> API -> Service -> Repository -> ORM`, con inyección de dependencias mediante `Depends()` y sesiones de base de datos por petición.

El módulo FinOps debe respetar esa estructura para no introducir una arquitectura paralela. La responsabilidad de cada capa queda así:

| Capa | Responsabilidad en Theia Officer | Responsabilidad FinOps |
|------|----------------------------------|-------------------------|
| Frontend | Presentar pantallas, estado cliente y llamadas HTTP | Dashboard de costes, estados de carga/error y filtros |
| API | Exponer endpoints autenticados | Contrato REST de costes, sincronización e insights |
| Service | Orquestar reglas de negocio | Agregación de costes, sincronización y análisis financiero |
| Repository | Encapsular acceso a datos | Consultas e idempotencia sobre datos de facturación |
| Model | Persistencia ORM | Entidades financieras y relaciones con organización/agentes |
| Integraciones | Conectores cloud y proveedores LLM | AWS Cost Explorer, Bedrock y proveedor LLM del CAIO |

### 5.2.2 Encaje del módulo FinOps

El módulo FinOps se apoya en entidades ya existentes de la plataforma: organizaciones, usuarios, credenciales cloud, agentes descubiertos, registros de auditoría y configuración LLM. La aportación del TFG consiste en añadir una vista financiera sobre esos elementos, no en sustituirlos.

La relación central es:

```text
Organización
  -> Credenciales AWS
  -> Agentes Bedrock descubiertos
  -> Datos de facturación
  -> Agregaciones FinOps
  -> Dashboard / análisis CAIO
```

Este encaje permite que la visibilidad de costes se mantenga aislada por organización, reutilice credenciales ya configuradas y pueda evolucionar hacia otros proveedores cloud sin cambiar el modelo conceptual principal.

### 5.2.3 Sistemas externos implicados

El análisis identifica tres sistemas externos relevantes:

| Sistema | Papel en el módulo | Observación de diseño |
|---------|--------------------|-----------------------|
| AWS Cost Explorer | Fuente de datos de coste | Proporciona coste histórico, no datos en tiempo real |
| Amazon Bedrock | Fuente de agentes IA | Permite relacionar costes con agentes descubiertos |
| Proveedor LLM del CAIO | Interpretación de datos | Analiza datos ya estructurados; no debe extraer datos de AWS |

La distinción es importante: el CAIO Virtual aporta valor interpretando patrones y recomendaciones, pero la lectura de datos estructurados de facturación debe ser determinista y auditable.

### 5.2.4 Flujo lógico de datos de facturación

El flujo propuesto para los datos financieros es:

```text
AWS Cost Explorer
  -> normalización de registros de coste
  -> persistencia idempotente en billing_data
  -> agregaciones por periodo, proveedor, servicio y agente
  -> exposición mediante API REST
  -> visualización en dashboard
  -> análisis textual opcional por el CAIO Virtual
```

Cost Explorer tiene una latencia propia de facturación, por lo que el módulo debe tratar los datos como históricos. No se debe prometer monitorización en tiempo real basada únicamente en esta fuente.

## 5.3 Análisis de casos de uso

### 5.3.1 Priorización de casos de uso FinOps

El capítulo 2 define los siguientes casos de uso nuevos:

| Prioridad | Casos de uso | Papel en el módulo |
|-----------|--------------|--------------------|
| Must | CU-07, CU-08 | Visibilidad base y detección de anomalías |
| Should | CU-09, CU-10, CU-11 | Optimización y análisis de eficiencia |
| Could | CU-12, CU-13 | Proyección y configuración de alertas |

CU-07 es la base operativa porque todos los casos posteriores dependen de disponer de datos de facturación normalizados. CU-08 también es Must, pero su diseño requiere histórico suficiente para evitar falsos positivos. CU-09 a CU-13 se analizan como ampliaciones coherentes del mismo modelo.

### 5.3.2 CU-07 — Dashboard consolidado de costes IA

CU-07 permite a un administrador o usuario regular consultar una vista consolidada de costes de IA. Según el capítulo 2, debe mostrar KPIs, desglose por agente y evolución temporal.

| Paso del caso de uso | Colaboración de análisis |
|----------------------|--------------------------|
| El usuario accede al dashboard | `CostsDashboardView` solicita datos al controlador de costes |
| El sistema obtiene costes del periodo | `BillingDashboardController` consulta `BillingData` |
| El sistema calcula KPIs | Se agregan costes por proveedor, agente y periodo |
| El sistema muestra tabla y gráficos | Vistas primitivas renderizan KPIs, tendencia y desglose |
| El usuario filtra | La vista actualiza parámetros de consulta |

Escenarios relevantes:

| Escenario | Comportamiento esperado |
|-----------|-------------------------|
| Hay datos sincronizados | Mostrar KPIs, tendencia temporal y desglose por agente |
| No hay datos | Mostrar estado vacío y sugerir sincronizar facturación |
| Solo hay datos de un proveedor | Mostrar el proveedor disponible sin forzar comparación multi-cloud |
| El CAIO no está disponible | Mostrar datos numéricos sin bloquear el dashboard |

### 5.3.3 CU-08 — Detección de anomalías de gasto

CU-08 detecta picos de gasto inusuales respecto al histórico. En análisis se identifica como un caso Must, pero dependiente de datos acumulados: sin varias semanas de registros reales, cualquier umbral estadístico sería arbitrario.

| Elemento | Análisis |
|----------|----------|
| Actor principal | CAIO Virtual / Actor Tiempo |
| Datos necesarios | Coste por agente, periodo y proveedor |
| Dependencia | Histórico suficiente para calcular línea base |
| Salida | Anomalía registrada y notificación al administrador |

La decisión de análisis es no tratar CU-08 como una simple pantalla adicional. Es un proceso periódico que debe reutilizar `BillingData`, aplicar una regla estadística documentada y registrar el resultado de forma auditable.

### 5.3.4 CU-09 a CU-13 — Casos de evolución

Los casos Should y Could se analizan para que el diseño de CU-07 no cierre puertas:

| Caso | Necesidad de datos | Dependencia principal |
|------|--------------------|-----------------------|
| CU-09 Analizar eficiencia | Coste + señales de uso | Requiere relacionar coste con utilización real |
| CU-10 Coste del CAIO Virtual | `LLMUsageLog` | Reutiliza logs de uso de proveedores LLM |
| CU-11 Optimizar modelo | Coste + modelo + calidad esperada | Requiere criterios de sustitución de modelos |
| CU-12 Proyectar tendencia | Serie temporal de costes | Requiere histórico estable |
| CU-13 Alertas de umbral | Coste por periodo + límites configurados | Requiere configuración persistente de umbrales |

Estos casos no necesitan el mismo nivel de secuencia que CU-07 en esta entrega, pero sí condicionan el diseño: la tabla de facturación debe ser temporal, multi-tenant, filtrable por proveedor y asociable a agentes.

### 5.3.5 Trazabilidad CU × pantalla × API × modelo

La siguiente tabla no afirma que todos los endpoints estén implementados. Define el contrato de diseño necesario para cubrir los casos de uso del módulo.

| Caso de uso | Pantalla / proceso | API de diseño | Modelo principal |
|-------------|--------------------|---------------|------------------|
| CU-07 Dashboard | `/costs` | `GET /billing/status`, `GET /billing/costs`, `GET /billing/costs/trend` | `BillingData`, `Agent` |
| CU-07 Desglose | `/costs` | `GET /billing/costs/breakdown` | `BillingData` |
| CU-07 Insights | `/costs` | `GET /billing/costs/insights` | `BillingData`, `LLMUsageLog` |
| CU-07 Sincronización | Acción administrativa | `POST /billing/sync` | `BillingData`, `AuditLog` |
| CU-08 Anomalías | Proceso periódico | `GET /billing/anomalies` o tarea interna | `BillingData` histórico |
| CU-09 Eficiencia | Vista de optimización | `GET /billing/efficiency` | `BillingData`, señales de uso |
| CU-10 Coste CAIO | Vista de coste plataforma | `GET /llm/costs` | `LLMUsageLog` |
| CU-11 Recomendaciones | Proceso CAIO | `GET /llm/optimization` | `LLMUsageLog`, configuración LLM |
| CU-12 Proyección | Dashboard / informe | `GET /billing/projection` | `BillingData` |
| CU-13 Alertas | Configuración | `POST /billing/alerts` | configuración de umbrales |