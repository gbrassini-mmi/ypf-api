# Pendientes

> Tareas que quedaron abiertas al cierre de una sesión, para retomar más
> adelante. Cuando un pendiente se resuelve, se mueve de "Abiertos" a
> "Resueltos" (no se borra, para mantener trazabilidad). Ver la sección
> "Mantenimiento de esta documentación" en `CLAUDE.md`.

## Abiertos

- [ ] (2026-09-30) Enviar a YPF el mapa Modbus del equipo y el listado de alarmas (provistos por Ingeniería) para que elijan tópicos/KPIs y alarmas -- se envía el 2026-09-30.
- [ ] (2026-09-30) Recibir de YPF la selección de tópicos y alarmas y cargarla en las tablas de whitelist -- depende del punto anterior. No bloquea el desarrollo (acordado 2026-09-30).
- [ ] (2026-09-30) Definir con YPF los límites del histórico: ventana máxima por request, granularidad (crudo o agregado), paginación y retención disponible -- quedó para la reunión con YPF.
- [ ] (2026-09-30) Obtener los rangos IP de salida de YPF (sandbox y prod) para la allowlist.
- [ ] (2026-09-30) Mapear en Aurora las tablas y columnas de origen de cada recurso (live, histórico, alarmas, desarenado, totalizadores).
- [ ] (2026-09-30) Relevar qué cálculos se delegan al backend interno (totalizadores de proceso, ¿otros?) y cómo se autentica la API contra él.
- [ ] (2026-09-30) Definir el canal seguro de entrega del `client_secret` a YPF y la política de rotación.
- [ ] (2026-09-30) Evaluar un log de auditoría de accesos (client_id, endpoint, equipo, IP, timestamp). No se incluyó en el alcance inicial, pero es probable que YPF lo valore por su foco en seguridad. Como mínimo, habilitar los access logs de API Gateway y los logs de WAF.
- [ ] (2026-09-30) Confirmar con YPF la zona horaria y el formato de fechas (propuesta: ISO-8601 UTC) y las unidades de cada tópico.
- [ ] (2026-09-30) Definir los valores concretos de rate limiting / throttling (rps y ráfaga) según la frecuencia de consulta que espera YPF.
- [ ] (2026-09-30) Definir los dominios (sandbox y prod) y dónde viven las tablas de configuración de la API.
- [ ] (2026-09-30) Definir tooling del repo (ORM/driver, lint, tests, CI/CD) al crear el primer código, y actualizar `CLAUDE.md`.
- [ ] (2026-09-30) Acordar con YPF la fecha de disponibilidad del sandbox (credenciales + Swagger), para que el equipo RTIC integre antes de la prueba funcional del 2026-10-23.
- [ ] (2026-09-30) Coordinar con YPF (Jorge, Rodrigo) la prueba de conexión y la validación de puertos: la API solo expone HTTPS por el 443.
- [ ] (2026-09-30) Acceso a cámaras de los equipos: fuera del alcance de esta API. Santiago consulta con Motomecánica; registrar el resultado.
- [ ] (2026-09-30) Confirmar la fecha de inicio que figura en la minuta anterior ("23 de septiembre"), que no coincide con el inicio de desarrollo del 2026-10-01.
- [ ] (2026-09-30) Confirmar si `services` pasa a ser el recurso raíz de la API y si el desarenado se expone como `dumps` (así quedó en el temario), y actualizar `docs/API-DESIGN.md` y `CLAUDE.md`.
- [ ] (2026-09-30) Esperar la respuesta de YPF por mail a las preguntas del temario (no se respondieron en la reunión).

## Resueltos

<!-- - [x] (YYYY-MM-DD abierto → YYYY-MM-DD resuelto) Descripción -- cómo se resolvió -->
