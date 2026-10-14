# Pendientes

> Tareas que quedaron abiertas al cierre de una sesión, para retomar más
> adelante. Cuando un pendiente se resuelve, se mueve de "Abiertos" a
> "Resueltos" (no se borra, para mantener trazabilidad). Ver la sección
> "Mantenimiento de esta documentación" en `CLAUDE.md`.

## Abiertos

- [ ] (2026-09-30) Recibir de YPF la selección de tópicos y alarmas y cargarla en las tablas de whitelist -- depende del punto anterior. No bloquea el desarrollo (acordado 2026-09-30).
- [ ] (2026-09-30) Definir con YPF los límites del histórico: ventana máxima por request, granularidad (crudo o agregado), paginación y retención disponible -- quedó para la reunión con YPF.
- [ ] (2026-09-30) Obtener los rangos IP de salida de YPF (sandbox y prod) para la allowlist.
- [ ] (2026-09-30) Mapear en Aurora las tablas y columnas de origen de cada recurso (live, histórico, alarmas, desarenado, totalizadores).
- [ ] (2026-09-30) Relevar qué cálculos se delegan al backend interno (totalizadores de proceso, ¿otros?) y cómo se autentica la API contra él.
- [ ] (2026-09-30) Definir el canal seguro de entrega del `client_secret` a YPF y la política de rotación.
- [ ] (2026-09-30) Evaluar un log de auditoría de accesos (client_id, endpoint, equipo, IP, timestamp). No se incluyó en el alcance inicial, pero es probable que YPF lo valore por su foco en seguridad. Como mínimo, habilitar los access logs de API Gateway y los logs de WAF.
- [ ] (2026-09-30) Definir los valores concretos de rate limiting / throttling (rps y ráfaga) según la frecuencia de consulta que espera YPF.
- [ ] (2026-09-30) Definir los dominios (sandbox y prod) y dónde viven las tablas de configuración de la API.
- [ ] (2026-09-30) Definir tooling del repo (ORM/driver, lint, tests, CI/CD) al crear el primer código, y actualizar `CLAUDE.md`.
- [ ] (2026-09-30) Coordinar con YPF (Jorge, Rodrigo) la prueba de conexión y la validación de puertos: la API solo expone HTTPS por el 443.
- [ ] (2026-09-30) Acceso a cámaras de los equipos: fuera del alcance de esta API. Santiago consulta con Motomecánica; registrar el resultado.
- [ ] (2026-09-30) Confirmar la fecha de inicio que figura en la minuta anterior ("23 de septiembre"), que no coincide con el inicio de desarrollo del 2026-10-01.
- [ ] (2026-10-14) YPF debe, antes del sandbox del 2026-10-21: IPs de salida, frecuencia esperada de consumo, canal de entrega y rotación de credenciales.
- [ ] (2026-10-14) YPF debe comentarios sobre la documentación de ciberseguridad enviada el 2026-10-07.
- [ ] (2026-10-14) `/live` devuelve todos los tópicos en valor crudo, sin multiplicador ni unidad: falta aplicar la whitelist de tópicos y las unidades (depende de la selección de YPF).
- [ ] (2026-10-14) La referencia v1 todavía marca "A definir con YPF" el rango máximo del histórico y los límites de paginación (hoy `limit` 1–1000, sin tope de rango). No figuró entre los pendientes de YPF del 2026-10-14: confirmar si se cierra internamente.
- [ ] (2026-10-14) La referencia v1 no incluye alarmas (activas/históricas) ni totalizadores, que están en el alcance original: confirmar estado.
- [ ] (2026-10-14) Sincronizar `CLAUDE.md`, `docs/API-DESIGN.md` y `docs/DECISIONS.md` con la implementación real que describe `docs/referencia-api-v1.html`: equipos por `serial`, visibilidad por cuentas/subcuentas con servicio activo (no la tabla `api_client_equipment`), `403` (no `404`) para equipos no visibles, scope `msuite/read` y permiso `DEVICE_READ`, rutas `well/{n}` vs `wells/{n,...}`, histórico paginado por página.


## Resueltos

<!-- - [x] (YYYY-MM-DD abierto → YYYY-MM-DD resuelto) Descripción -- cómo se resolvió -->

- [x] (2026-09-30 abierto → 2026-10-14 resuelto) Enviar a YPF el mapa Modbus del equipo y el listado de alarmas (provistos por Ingeniería) para que elijan tópicos/KPIs y alarmas -- se envía el 2026-09-30. -- enviado el 2026-09-30.
- [x] (2026-09-30 abierto → 2026-10-14 resuelto) Confirmar con YPF la zona horaria y el formato de fechas (propuesta: ISO-8601 UTC) y las unidades de cada tópico. -- la referencia v1 usa ISO 8601 en UTC. Las unidades siguen pendientes (el live devuelve valor crudo sin unidad).
- [x] (2026-09-30 abierto → 2026-10-14 resuelto) Acordar con YPF la fecha de disponibilidad del sandbox (credenciales + Swagger), para que el equipo RTIC integre antes de la prueba funcional del 2026-10-23. -- sandbox el miércoles 2026-10-21.
- [x] (2026-09-30 abierto → 2026-10-14 resuelto) Confirmar si `services` pasa a ser el recurso raíz de la API y si el desarenado se expone como `dumps` (así quedó en el temario), y actualizar `docs/API-DESIGN.md` y `CLAUDE.md`. -- la implementación (referencia v1) mantiene `/v1/equipment/{serial}` y `sanding-events`.
- [x] (2026-09-30 abierto → 2026-10-14 resuelto) Esperar la respuesta de YPF por mail a las preguntas del temario (no se respondieron en la reunión). -- reemplazado por los pendientes concretos del 2026-10-14.
