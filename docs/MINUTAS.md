# Minutas de reunión con YPF

> Registro de las reuniones con YPF sobre la API. La más nueva va al final.

---

## Reunión inicial (anterior al 2026-09-30)

**Participantes:** Santiago Landa (SL), Germán Brassini (GB), Andrés Finessi (AF), Rafael Ibarra (RI), Jorge Vázquez (JV), Mercys Acuña (MA), Jorge Jeldres (JJ), Rodrigo Ruiz (RR).

**Objetivo:** construir una interfaz de software entre MMA e YPF para transmitir los datos de gestión de desarenadores.

**Temas:**
- **Integración de la API:** se acordó una API específica para que el equipo **RTIC** de YPF consuma datos operativos. Se prioriza una **prueba funcional antes del 23 de octubre** y no se reutilizan interfaces no adaptadas al contexto de YPF.
- **Variables y alarmas:** MMA comparte un inventario de variables, alertas y alarmas, y YPF elige lo relevante para la operación remota y la seguridad.
- **Accionamiento remoto y seguridad:** se analizó cómo llevar señales de emergencia y condiciones críticas a una sala remota. Se separa la adquisición de datos de la ejecución de acciones y se priorizan los indicadores de seguridad del personal.
- **Cámaras:** se evaluó sumar el acceso a las cámaras de los equipos para la sala remota. Santiago consulta con Motomecánica las condiciones de acceso a la plataforma del fabricante.
- **Coordinación:** fecha de inicio registrada en la minuta como "23 de septiembre" (a confirmar). Objetivo: prueba funcional el 23 de octubre.

**Tareas de seguimiento:**
- Compartir el listado de variables y la documentación de alertas y alarmas. (Germán)
- Revisar el listado, elegir variables y alarmas y enviar feedback. (Rodrigo, Mercys, Jorge)
- Compartir la documentación de la API: endpoints, consulta en vivo e histórica, autenticación aplicación a aplicación. (Germán)
- Diseñar el cliente consumidor y coordinar una prueba de conexión, incluyendo validación de puertos y requisitos de ciberseguridad, para el test funcional antes del 23/10/2026. (Jorge, Rodrigo)
- Consultar con Motomecánica si se puede habilitar el acceso a las cámaras y comunicar el resultado. (Santiago)

**Estado al 2026-09-30 (según Germán):** el alcance y los pasos de desarrollo se definieron el jueves y viernes de la semana anterior. Hoy cierra el sprint anterior y se envía el detalle de alarmas y tópicos. El desarrollo empieza el 2026-10-01. La selección de tópicos y alarmas por parte de YPF **no bloquea** el desarrollo.

## 2026-09-30 -- Reunión de detalle

Temario preparado: https://claude.ai/artifact/6DYk4vtyypcyad7fBgCvRJ (estado de tareas, tiempos, seguridad, documentación y temas a definir). Reunión realizada. Las preguntas del temario (IPs de salida, límites del histórico, frecuencia de consulta, credenciales, fechas/unidades, fecha del sandbox) **no se respondieron en la reunión**: YPF las responde por mail. Por mail se envían también el temario, el mapa Modbus y el detalle de alarmas.

## 2026-10-14 -- Reunión de seguimiento

Temario: https://claude.ai/artifact/UGzmtE5ieRrVbVeXph5EYd

**Estado al inicio de la reunión:**
- Listado de variables, alarmas y alertas: enviado a YPF el 2026-09-30. Falta la selección final de YPF. No bloquea, pero se necesita pronto para cerrar el desarrollo.
- Documentación de la API: se presenta y comparte hoy en formato documento (`docs/referencia-api-v1.html`). El Swagger se entrega con el sandbox.
- Ciberseguridad: documentación de seguridad e instrucciones de conexión enviadas el 2026-10-07. Esperando comentarios de YPF; si hay dudas, reunión con los desarrolladores.
- Sandbox: disponible el miércoles 2026-10-21.
- Cámaras: sigue a cargo de Santiago; ya se comunicó a los responsables del acceso.

**YPF debe:** IPs de salida, frecuencia esperada de consumo, entrega y rotación de credenciales (las tres antes del 2026-10-21), selección final de variables y alarmas, comentarios de ciberseguridad.
