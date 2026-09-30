# CLAUDE.md -- YPF API (API de consumo de datos de desarenadores)

> Documento de contexto para agentes de IA. Se mantiene actualizado a medida
> que el proyecto evoluciona -- no es una foto fija del día en que se creó.

## Resumen

API REST de **consumo (pull)** que Digito expone a **YPF** para que sus
sistemas lean los datos de los equipos desarenadores. Digito **no emite**
datos hacia YPF: YPF consulta la API cuando lo necesita. El consumidor es el
equipo **RTIC** de YPF (sala de operación remota). Cubre:

- **Dato en vivo**: el último valor emitido por el equipo en cada tópico.
- **Histórico**: serie temporal de cada tópico.
- **Alarmas**: activas en el momento de la consulta e histórico de alarmas generadas.
- **Desarenado**: eventos de desarenado ocurridos en cada pozo de cada equipo.
- **Totalizadores**: diarios y de proceso (servicio completo).

YPF **no recibe todos los tópicos ni todas las alarmas**: recibe solo los que
elige. La selección se guarda en tablas de configuración (whitelist, ver
"Modelos de datos").

**Estado:** repositorio recién creado (2026-09-30), todavía **sin código**.
Todo lo de abajo es el diseño acordado; se actualiza a medida que se implementa.
El desarrollo empieza el 2026-10-01 y el objetivo es una **prueba funcional con YPF el 2026-10-23**.

**Fuera de alcance:** accionamiento remoto (la API solo adquiere datos, nunca ejecuta
acciones sobre los equipos) y acceso a cámaras (depende de Motomecánica).

## Stack tecnológico

- **Backend:** NestJS (TypeScript). Documentación con `@nestjs/swagger` (OpenAPI 3).
- **Base de datos:** AWS Aurora (Postgres), acceso **solo lectura** a través del
  **reader endpoint** (réplica de lectura) con un usuario de base read-only.
  ORM/driver: *a definir al arrancar el código* (alinear con los otros repos de Digito).
- **Infraestructura / cloud:** AWS:
  - **ECS Fargate** corre el servicio NestJS (en subredes privadas).
  - **API Gateway (REST API)** es la única puerta de entrada, con conexión privada a Fargate por **VPC Link + NLB**.
  - **Amazon Cognito** (User Pool con resource server): OAuth2 `client_credentials` (autenticación entre aplicaciones).
  - **AWS WAF** asociado al stage de API Gateway.
  - **IaC:** AWS CDK en TypeScript, dentro de este repo.
- **Herramientas:** *a definir con el primer commit de código* (lint, formato, tests, CI/CD).

## Estructura del repositorio

Propuesta; se crea a medida que se implementa. Actualizar este árbol cuando cambie:

```
ypf-api/
├── CLAUDE.md              # este archivo
├── docs/                  # toda la documentación .md (ver "Mantenimiento")
│   ├── API-DESIGN.md      # endpoints, seguridad, documentación (material para YPF)
│   ├── MINUTAS.md         # minutas de reuniones con YPF
│   ├── HISTORY.md
│   ├── DECISIONS.md
│   └── PENDANTS.md
├── src/                   # app NestJS (un módulo por dominio, ver abajo)
├── infra/                 # app CDK (API GW, WAF, Cognito, ECS, VPC Link)
└── openapi/openapi.json   # spec OpenAPI exportado y versionado (se comparte con YPF)
```

## Arquitectura

```
Sistemas YPF
   │ 1) POST /oauth2/token (client_id + client_secret, grant client_credentials)
   ▼
Cognito (dominio del User Pool) ──► access_token (JWT, corta duración)
   │
   │ 2) GET /v1/...  Authorization: Bearer <token>
   ▼
AWS WAF (reglas administradas + regla rate-based + allowlist IP de YPF)
   ▼
API Gateway REST (resource policy con IPs de YPF, Cognito authorizer, throttling)
   ▼ VPC Link + NLB (privado)
ECS Fargate · NestJS  ──► Aurora reader endpoint (usuario read-only)
                      └─► Backend interno Digito (solo cálculos, p. ej. totalizadores)
```

- **Solo lectura, siempre.** La API no escribe nada en la plataforma. Todos los endpoints son `GET` (salvo el token de Cognito, que no es de esta API).
- **Fuente de datos:** todo se lee de **Aurora**. El **backend interno** se usa solo para cálculos que ya resuelve (p. ej. totalizadores de proceso). Todavía hay que relevar qué cálculos exactos (ver `docs/PENDANTS.md`).
- **Autorización por equipo:** el `client_id` del token se mapea a los equipos asignados (tabla `api_client_equipment`). **Cada consulta filtra por esa asignación.** Un equipo no asignado responde `404`, no `403`, para no revelar que existe.
- **Filtrado por whitelist:** tópicos y alarmas se filtran contra las tablas de configuración antes de responder. Un tópico o alarma fuera de la whitelist **nunca** sale en ninguna respuesta (live, histórico ni alarmas).
- **Versionado en la URL:** todas las rutas llevan el prefijo `/v1`. Un cambio incompatible implica `/v2`, conviviendo con `/v1` hasta que YPF migre.
- **Ambientes:** `sandbox` (QA para que YPF integre, con su propio client de Cognito y sus propias credenciales) y `prod`.
- **Alcance del producto:** API **exclusiva para YPF** (ver `docs/DECISIONS.md`). La asignación por `client_id` existe por seguridad, no para venderla a otros clientes.

## Componentización / Módulos clave

Módulos NestJS previstos (uno por dominio, bajo `src/`):

| Módulo | Responsabilidad |
|---|---|
| `auth` | Validación del JWT de Cognito (defensa en profundidad, además del authorizer de API GW), extracción del `client_id`, guard de equipos asignados. |
| `equipment` | Lista de equipos y pozos asignados al cliente. |
| `live` | Último valor por tópico habilitado de un equipo. |
| `history` | Serie temporal de un tópico (límites de rango y paginación: a definir con YPF). |
| `alarms` | Alarmas activas e histórico, filtradas por la whitelist de alarmas. |
| `sanding` | Eventos de desarenado por pozo. |
| `totalizers` | Totalizadores diarios y de proceso (servicio completo). |
| `config` | Lectura de las whitelists de tópicos/alarmas y de la asignación de equipos. |

## Modelos de datos / esquemas

- **Datos de la plataforma** (tópicos, alarmas, desarenado, totalizadores): tablas existentes en Aurora. *Hay que mapear qué tablas y columnas son* (pendiente).
- **Configuración propia de la API** (tablas nuevas, nombres tentativos):
  - `api_client_equipment` (`client_id`, `equipment_id`): qué equipos ve cada cliente de Cognito.
  - `api_enabled_topic` (`topic_key`, alias/unidad expuestos): tópicos/KPIs que YPF eligió a partir del mapa Modbus.
  - `api_enabled_alarm` (`alarm_code`, descripción expuesta): alarmas que YPF eligió.
  - Las whitelists se cambian **sin redeploy**. Como la API es read-only, la escritura de estas tablas se hace por migración o script administrativo, nunca desde la API.

## Scripts y comandos

Todavía no hay código. Completar esta tabla cuando exista `package.json`.

| Comando | Qué hace |
|---|---|
| — | — |

## Convenciones de código

- TypeScript en modo `strict`. DTOs de respuesta con decoradores de `@nestjs/swagger` en **todos** los campos: la doc de YPF se genera desde el código.
- Todo endpoint nuevo: `GET`, bajo `/v1`, protegido por el guard de equipos, filtrado por whitelist y documentado en Swagger con ejemplos.
- Fechas en ISO-8601 UTC en todas las respuestas (confirmar con YPF, ver pendientes).
- Nunca exponer IDs internos ni campos de la plataforma que no estén en el contrato documentado.
- Errores con un formato uniforme (`statusCode`, `error`, `message`), sin stack traces ni SQL.

## Variables de entorno / configuración

Nombres tentativos; ajustar al implementar. Los secretos van en AWS Secrets Manager, nunca en el repo.

- `DB_READER_HOST`, `DB_PORT`, `DB_NAME`, `DB_SECRET_ARN`: conexión al reader de Aurora (credenciales del usuario read-only en Secrets Manager).
- `COGNITO_USER_POOL_ID`, `COGNITO_REGION`, `COGNITO_ISSUER`: validación del JWT.
- `INTERNAL_BACKEND_URL`: backend interno para cálculos.
- `APP_ENV`: `sandbox` | `prod`.

## Integraciones externas

- **Amazon Cognito:** emisión de tokens `client_credentials` para YPF.
- **AWS Aurora:** fuente de todos los datos (solo lectura).
- **Backend interno de Digito:** cálculos (totalizadores de proceso, etc.).
- **AWS WAF / API Gateway / ECS Fargate:** exposición y ejecución.

## Documentación relacionada

- Diseño de la API, seguridad y documentación (material para YPF): [`docs/API-DESIGN.md`](docs/API-DESIGN.md)
- Minutas de reuniones con YPF: [`docs/MINUTAS.md`](docs/MINUTAS.md)
- Temario de la reunión del 2026-09-30 (artifact): https://claude.ai/artifact/6DYk4vtyypcyad7fBgCvRJ
- Historial de sesiones: [`docs/HISTORY.md`](docs/HISTORY.md)
- Decisiones técnicas: [`docs/DECISIONS.md`](docs/DECISIONS.md)
- Pendientes: [`docs/PENDANTS.md`](docs/PENDANTS.md)

## Mantenimiento de esta documentación

- Todo archivo `.md` de este repositorio, **excepto este `CLAUDE.md`**, vive
  dentro de `docs/` en la raíz.
- `CLAUDE.md` se actualiza en la misma sesión en que cambia algo estructural
  del proyecto (stack, arquitectura, estructura de carpetas, módulos
  nuevos). No se deja para "después".
- **Al final de cada sesión de trabajo**, un agente que haya hecho cambios
  en este repositorio debe:
  1. Agregar una entrada en `docs/HISTORY.md` con las tareas realizadas en
     esa sesión.
  2. Agregar en `docs/DECISIONS.md` cualquier decisión técnica relevante
     que se haya tomado (cuando hubo más de una opción válida y se optó por
     una).
  3. Actualizar `docs/PENDANTS.md` si quedó algo abierto para retomar más
     adelante, o si se resolvió algo que estaba pendiente.
- El formato exacto de cada entrada está documentado dentro de cada uno de
  esos tres archivos.
