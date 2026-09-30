# Diseño de la API: endpoints, seguridad y documentación

> Material de referencia para el equipo de Digito y para compartir con YPF.
> Los endpoints son una **propuesta inicial** (2026-09-30), sujeta a lo que
> se defina con YPF. Cuando se implementen, la fuente de verdad pasa a ser
> el spec OpenAPI generado (`openapi/openapi.json`).

## 1. Modelo de consumo

- **Pull:** YPF consulta la API. Digito no envía datos por push, webhooks ni colas.
- **Solo lectura:** todos los endpoints son `GET`.
- **Base URL:** `https://<dominio>/v1` (un dominio para `sandbox` y otro para `prod`, a definir).
- **Formato:** JSON. Fechas en ISO-8601 UTC (a confirmar con YPF).

## 2. Seguridad

Capas de control, en el orden en que las atraviesa un request:

| # | Capa | Qué hace |
|---|---|---|
| 1 | **TLS 1.2+** | Todo el tráfico va cifrado en tránsito. No hay endpoint HTTP plano. |
| 2 | **Allowlist de IPs de YPF** | Solo se aceptan requests desde los rangos IP de salida que informe YPF. Se aplica en WAF (IP set) y en la resource policy de API Gateway. |
| 3 | **AWS WAF** | Reglas administradas de AWS (Core rule set, Known bad inputs, IP reputation) y una regla rate-based por IP. |
| 4 | **Rate limiting / throttling** | Límites de tasa y ráfaga en el stage y en cada método de API Gateway. |
| 5 | **OAuth2 `client_credentials` (Cognito)** | Autenticación entre aplicaciones. YPF recibe un `client_id` y un `client_secret` por ambiente, pide un `access_token` (JWT de corta duración) al endpoint `/oauth2/token` de Cognito y lo manda como `Authorization: Bearer <token>`. API Gateway valida el token con un Cognito authorizer. |
| 6 | **Validación en la aplicación** | El servicio vuelve a validar el JWT (firma, issuer, expiración), resuelve el `client_id` y restringe la consulta a los equipos asignados. |
| 7 | **Whitelist de datos** | Solo salen los tópicos y alarmas que YPF eligió. |
| 8 | **Red privada** | El servicio corre en Fargate en subredes privadas, sin IP pública. Solo se llega desde API Gateway (VPC Link). |
| 9 | **Base en solo lectura** | Conexión al reader endpoint de Aurora con un usuario que solo tiene `SELECT`. Aunque la API tuviera una vulnerabilidad, no podría modificar datos. |
| 10 | **Gestión de secretos** | Las credenciales de base y otros secretos van en AWS Secrets Manager. Nada sensible en el código ni en variables planas. |

**Credenciales de YPF:** el `client_secret` se entrega por un canal seguro (a
definir; nunca por mail en texto plano). La política de rotación está pendiente.

**Aislamiento sandbox / prod:** ambientes separados, cada uno con su propio app client de Cognito y sus credenciales. Un token de sandbox no sirve en prod.

## 3. Endpoints propuestos

`{equipmentId}` es siempre un equipo asignado al cliente. Si no lo está, la respuesta es `404`.

| Método y ruta | Descripción |
|---|---|
| `GET /v1/equipment` | Equipos asignados a YPF. |
| `GET /v1/equipment/{equipmentId}` | Detalle del equipo y sus pozos. |
| `GET /v1/equipment/{equipmentId}/live` | Último valor emitido de cada tópico habilitado (valor, unidad, timestamp). |
| `GET /v1/equipment/{equipmentId}/topics` | Tópicos habilitados para el equipo (catálogo). |
| `GET /v1/equipment/{equipmentId}/topics/{topicKey}/history?from=&to=` | Histórico de un tópico. Límites y paginación: a definir con YPF. |
| `GET /v1/equipment/{equipmentId}/alarms/active` | Alarmas activas (solo las habilitadas). |
| `GET /v1/equipment/{equipmentId}/alarms/history?from=&to=` | Alarmas generadas en el período (inicio, fin, código, descripción). |
| `GET /v1/equipment/{equipmentId}/wells/{wellId}/sanding-events?from=&to=` | Eventos de desarenado del pozo. |
| `GET /v1/equipment/{equipmentId}/totalizers/daily?date=` | Totalizadores diarios. |
| `GET /v1/equipment/{equipmentId}/totalizers/service` | Totalizadores de proceso (servicio completo). |

Códigos de respuesta comunes: `200`, `400` (parámetros inválidos), `401` (token ausente o inválido), `403` (IP no permitida o scope insuficiente), `404` (recurso inexistente o no asignado), `429` (rate limit superado), `5xx`.

## 4. Documentación (Swagger / OpenAPI)

- La doc se **genera desde el código** con `@nestjs/swagger`: cada DTO y endpoint lleva descripciones y ejemplos.
- **Swagger UI** se sirve en `/docs`, detrás de la **misma allowlist de IPs**. No es pública.
- El **spec `openapi.json`** se exporta y se versiona en el repo (`openapi/openapi.json`). Se le envía a YPF en cada versión para que genere sus clientes o integre sin depender del UI.
- Cada cambio de contrato queda registrado en un changelog de la API (a crear en `docs/`).

## 5. Qué necesita YPF para empezar a integrar

1. Rangos IP de salida (sandbox y prod).
2. Lista de tópicos/KPIs elegidos (sobre el mapa Modbus que envía Digito).
3. Lista de alarmas elegidas (sobre el listado de alarmas que envía Digito).
4. Definición de límites y formato del histórico (ventana máxima, granularidad, paginación).
5. Credenciales de sandbox (las entrega Digito) y la URL del Swagger.
