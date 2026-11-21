# 08 · RÚBRICA, PRUEBAS DE PERMISOS Y GUION DE DEMO

## 1. Rúbrica (100 pts) → dónde se cumple
| Bloque | Pts | Requisito | Dónde / HU | Cómo se demuestra |
|---|---|---|---|---|
| Base de datos | 10 | MySQL (no MariaDB), Flyway, ≥5 tablas, N:M | `V1__esquema.sql`, `usuario_roles` · HU-01 | `mysql --version`, tabla `flyway_schema_history` |
| | | Datos de prueba 10-15 por tabla principal | `DataSeeder` · HU-01 | contar filas |
| | | Diagrama ER en README y `.sql` de respaldo | HU-32 | README y `database/rutaoax_backup.sql` |
| Backend | 15 | @RestController/@RequestMapping, DTOs, Bean Validation, @ControllerAdvice | HU-02 a HU-26 | Bruno 422/404/409 |
| | | Spring Security + JWT, login, registro, logout con invalidación, recuperación | HU-03 a HU-06, HU-08 | Bruno `Auth/*` |
| | | JPA con relaciones y sin N+1 | entidades LAZY + JOIN FETCH | revisar logs SQL de un listado |
| Frontend | 15 | Rutas protegidas, validaciones bajo inputs, paginación y filtros del servidor | HU-07, HU-10, HU-12, HU-30 | demo + pestaña Network (`?page=&q=`) |
| | | Navbar con nombre, correo, avatar y logout; modales; sin alert/confirm; responsivo; loading y error de red | HU-07, HU-30 | `grep -r "alert(" src` vacío; vista móvil |
| Usuarios y roles | 10 | ≥3 niveles, Spring Security por rol, rutas React por rol, admin documentado | HU-04, HU-07, HU-08, HU-29 | login con cada rol |
| Comunicación | 10 | Correo real con Postfix + JavaMailSender (SPF y DKIM) | HU-06, HU-26 | correo en bandeja con SPF/DKIM PASS |
| | | SMS real desde la app | HU-17 | SMS al celular al iniciar ruta |
| | | WhatsApp real desde la app | HU-18 | WhatsApp al marcar entregado |
| Figma | 10 | Prototipo navegable, paleta justificada, logo original | HU-33 | enlace en README |
| GitHub | 12 | README completo, commits de ambos, repo público | HU-32 | historial de commits |
| GitHub Projects | 12 | Tablero activo y público, tareas reales en Backlog/In Progress/Done durante todo el proyecto | HU-00 y ritual diario | historial del tablero |
| Bruno | 6 | Colección en el repo: login, token, uso del token, error a propósito | HU-31 | abrir la colección |
| Seguridad / despliegue (condiciones) | — | Secretos fuera del repo, `.example`, BCrypt, JAR con systemd, frontend estático, Nginx, HTTPS con Certbot, URL en README | HU-01, HU-09 | `https://rutaoax.devs8.tech`, `systemctl status` |

## 2. Matriz de permisos (HU-29)
Ejecutar con Bruno usando el token de cada rol. ✅ = permitido, ⛔ = debe dar 403 (o 404 si el recurso es de otro).
| Endpoint | ADMIN | NEGOCIO | REP. empresa | REP. independiente | CLIENTE |
|---|---|---|---|---|---|
| GET `/admin/usuarios` | ✅ | ⛔ | ⛔ | ⛔ | ⛔ |
| GET `/vehiculos` | ✅ | ✅ (solo suyos) | ⛔ | ⛔ | ⛔ |
| POST `/equipo` | ⛔ (usar admin) | ✅ | ⛔ | ⛔ | ⛔ |
| GET `/pedidos` | ✅ todos | ✅ suyos | ✅ de su recorrido | ✅ los que tomó | ✅ suyos |
| POST `/pedidos/asignar` | ⛔ (usar la cuenta del negocio) | ✅ | ⛔ | ⛔ | ⛔ |
| PATCH `/pedidos/{id}/estado` | ✅ | ⛔ | ✅ su parada | ✅ su parada | ⛔ |
| GET `/disponibles` | ✅ | ⛔ | ⛔ | ✅ | ⛔ |
| POST `/disponibles/{id}/tomar` | ⛔ | ⛔ | ⛔ | ✅ | ⛔ |
| GET `/recorridos/hoy` | ⛔ | ⛔ | ✅ | ✅ | ⛔ |
| GET `/mensajes/hilos` | ⛔ | ✅ | ⛔ | ✅ | ✅ |
| GET `/pedidos/{id}/seguimiento` | ✅ | ✅ suyos | ⛔ | ⛔ | ✅ suyos |
Además: ningún rol ve datos de otro negocio, otro cliente u otro repartidor, y ninguna respuesta incluye `passwordHash`.

## 3. Guion de la demo en vivo (≈12 minutos)
Antes de empezar: cambiar el teléfono de `cliente1` por tu número, unirte de nuevo al sandbox de WhatsApp, abrir el sitio en una ventana normal y otra privada, y tener el celular a la vista.
1. **Administrador** (`admin@rutaoax.com`): usuarios con paginación y filtro, crear una cuenta, ver el log de notificaciones.
2. **Negocio** (`negocio1@rutaoax.com`): panel, crear un pedido para el correo de `cliente1`, asignarlo a `repartidor1` con un vehículo (mostrar el cupo), vehículos y equipo.
3. **Repartidor de empresa** (`repartidor1@rutaoax.com`, ventana privada): su ruta, **iniciar recorrido → llega el SMS**, parada actual, **marcar entregado → llega el WhatsApp**.
4. **Cliente** (`cliente1@rutaoax.com`): seguimiento actualizado y envío de un mensaje sobre el pedido; el negocio lo recibe en su bandeja con "Pedido #".
5. **Independiente** (`independiente1@rutaoax.com`): entregas disponibles, tomar una, mapa con la ruta, mensajes directos del cliente.
6. **Seguridad:** intentar abrir la URL de otro rol (pantalla de "No tienes permiso"), cerrar sesión y mostrar que el token viejo da 401 en Bruno.
7. **Correo:** pedir recuperación de contraseña y mostrar el correo en la bandeja (SPF y DKIM PASS).
8. **Repositorio:** README con ER, tablero de Projects con historial, commits de ambos, colección Bruno, Figma.

## 4. Lista final antes de entregar
- [ ] `https://rutaoax.devs8.tech` responde con candado válido y el servidor sigue arriba tras un reinicio
- [ ] Los 5 usuarios demo inician sesión en producción
- [ ] SMS, WhatsApp y correo llegan de verdad (probados el mismo día de la entrega)
- [ ] `application.properties` NO está en el repositorio; `application.properties.example` sí
- [ ] README con: ER, instalación, credenciales de cada rol, URL del sitio y de la API, enlaces a Figma y a Projects
- [ ] `database/rutaoax_backup.sql` actualizado
- [ ] Carpeta `bruno/` en el repo y colección corriendo en verde
- [ ] Repo público; commits de ambos integrantes; tablero público con tarjetas repartidas en las columnas
- [ ] Cero `alert(` / `confirm(` en `frontend/src`
- [ ] Cada integrante puede explicar su parte sin ayuda de la IA
