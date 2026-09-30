# Decisiones técnicas

> Registro de decisiones técnicas relevantes tomadas en este repositorio.
> No todo merece una entrada acá: se anota cuando hubo más de una opción
> válida y se optó por una, o cuando la decisión condiciona sesiones
> futuras. Ver la sección "Mantenimiento de esta documentación" en
> `CLAUDE.md`.

Formato de cada entrada (estilo ADR liviano):

```markdown
## YYYY-MM-DD -- Título de la decisión

**Contexto:** qué problema o disyuntiva llevó a esta decisión.
**Decisión:** qué se resolvió hacer.
**Alternativas consideradas:** qué otras opciones se evaluaron y por qué no.
**Consecuencias:** qué implica esto hacia adelante (trade-offs, deuda técnica, etc.).
```

---

## 2026-09-30 -- Modelo pull, solo lectura

**Contexto:** YPF necesita datos de los equipos en sus sistemas.
**Decisión:** API de consumo (pull). Digito no emite datos a YPF y la API no tiene endpoints de escritura.
**Alternativas consideradas:** push por webhooks, colas o MQTT. Se descartó por acuerdo con YPF: pull los deja controlar la frecuencia y simplifica la seguridad.
**Consecuencias:** la frecuencia de consulta la decide YPF, así que hacen falta rate limiting y límites en las consultas históricas.

## 2026-09-30 -- NestJS en ECS Fargate detrás de API Gateway

**Contexto:** hay que elegir runtime y forma de exposición.
**Decisión:** NestJS (TypeScript) en ECS Fargate, en subredes privadas, expuesto solo por API Gateway **REST API** con VPC Link + NLB.
**Alternativas consideradas:** NestJS en Lambda (cold starts y manejo de conexiones a Aurora), Lambdas sueltas (Swagger a mano), FastAPI (se aleja del stack del equipo). Se eligió REST API y no HTTP API porque WAF y las resource policies (allowlist de IPs) solo están disponibles en REST API.
**Consecuencias:** servicio siempre encendido (costo fijo) y pool de conexiones estable hacia Aurora. Suma un NLB y un VPC Link a la infraestructura.

## 2026-09-30 -- Seguridad: Cognito client_credentials + WAF + rate limiting + allowlist IP

**Contexto:** YPF presta mucha atención a la seguridad del acceso a los datos.
**Decisión:** OAuth2 `client_credentials` con Cognito, AWS WAF (reglas administradas + rate-based), throttling en API Gateway y allowlist de IPs de YPF (WAF + resource policy). Además, en la app: validación del JWT y restricción a los equipos asignados al `client_id`.
**Alternativas consideradas:** mTLS (no se adopta por ahora: suma gestión de certificados). Scopes OAuth por recurso (no se adoptan por ahora: con un solo consumidor no aportan tanto). Las dos opciones quedan disponibles si YPF las pide.
**Consecuencias:** YPF tiene que informar sus IPs de salida antes de integrar. Cualquier cambio de IP de su lado requiere un cambio de configuración nuestro.

## 2026-09-30 -- Datos desde Aurora (reader, read-only), backend interno solo para cálculos

**Contexto:** los datos viven en la plataforma existente.
**Decisión:** todo se lee del reader endpoint de Aurora con un usuario de base de solo lectura. El backend interno se usa únicamente para cálculos que ya resuelve (p. ej. totalizadores de proceso).
**Alternativas consideradas:** conectarse al writer del cluster (compite con la carga de la plataforma) o ir todo por el backend interno (más acoplamiento y latencia).
**Consecuencias:** la API depende del esquema de Aurora de la plataforma. Un cambio de esquema allá puede romper la API, así que hay que coordinarlo.

## 2026-09-30 -- Whitelist de tópicos y alarmas en tablas de la base

**Contexto:** YPF quiere solo un subconjunto de los tópicos y de las alarmas.
**Decisión:** tablas de configuración (`api_enabled_topic`, `api_enabled_alarm`) más la asignación de equipos (`api_client_equipment`), editables sin redeploy.
**Alternativas consideradas:** archivo de configuración en el repo (cada cambio implica deploy) y SSM/AppConfig (otra fuente de configuración para mantener).
**Consecuencias:** hay que definir dónde viven esas tablas (schema propio en Aurora; la API se conecta al reader, así que la escritura se hace por migración o script) y dejar trazabilidad de sus cambios.

## 2026-09-30 -- API exclusiva para YPF

**Contexto:** se evaluó diseñarla como producto multi-cliente.
**Decisión:** API exclusiva para YPF. Igual se mantiene el mapeo `client_id` → equipos, por seguridad.
**Alternativas consideradas:** API genérica con YPF como primer cliente.
**Consecuencias:** sumar otro cliente en el futuro puede requerir refactor (p. ej. whitelists por cliente).

## 2026-09-30 -- Documentación Swagger protegida + spec versionado, versión en la URL, ambientes sandbox y prod

**Contexto:** YPF necesita documentación para desarrollar su integración en paralelo, y un entorno para probar.
**Decisión:** Swagger UI en `/docs` detrás de la allowlist de IPs y `openapi.json` versionado en el repo. Prefijo `/v1` en todas las rutas. Ambientes `sandbox` y `prod`, con credenciales separadas. Infraestructura con AWS CDK (TypeScript) en este repo.
**Alternativas consideradas:** Swagger público (expone la superficie de la API), solo enviar el spec (peor experiencia de integración), Terraform o infra manual.
**Consecuencias:** hay que mantener el spec exportado al día en cada release.
