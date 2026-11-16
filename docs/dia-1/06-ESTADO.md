# 06 · ESTADO ACTUAL (actualizar al cerrar cada tarea y cada día)

Última actualización: ________ · Día: 1 · Parte 1 entregada (HU-01 y HU-02) · Quién: ________

## Resumen
Fase actual: **DÍA 1 · Parte 1 de 2** (Parte 1 = HU-01 y HU-02; Parte 2 = HU-03 y HU-04).
- Existe el **código de la Parte 1** del backend (esqueleto Spring Boot 3.5.7, Flyway, 11 entidades, repositorios, seeders y errores globales). **Falta que se ejecute y se verifique** con `docs/dia-1/EJECUTAR-PARTE-1.md`; hasta entonces las HU se quedan en 🟨.
- Siguen disponibles: diseños HTML/CSS (`diseno/`), documentos `00` a `08`, SQL `V1__esquema.sql`, colección Bruno (`bruno/`) y `docs/dia-1/` (interfaces compartidas y tareas del día).
- Frontend, despliegue y los endpoints de autenticación **todavía no existen**.
- Interfaces (nombres de clases y firmas) congeladas en `docs/dia-1/01-INTERFACES-COMPARTIDAS.md`. No cambiarlas sin anotarlo en "Decisiones".

## URLs y accesos
- Sitio: https://rutaoax.devs8.tech · API: https://rutaoax.devs8.tech/api · VPS: 179.197.84.34 · Backend local: http://localhost:9250/api · Frontend local: http://localhost:5173
- Repo: ________ · Tablero: ________ · Figma: ________
- Credenciales demo: admin `admin@rutaoax.com` / `Admin#2026`; resto `Ruta#2026` (negocio1@, negocio2@, repartidor1@ a repartidor4@, independiente1@, independiente2@, cliente1@ a cliente4@ … @rutaoax.com).
- MySQL local: base `rutaoax`, usuario `rutaoax_app`.

## Plan del Día 1 (tareas de `docs/dia-1/02-TAREAS-DIA-1.md`)
Estados: ⬜ pendiente · 🟨 en curso o por verificar · ✅ hecha y verificada · ❌ bloqueada
| Tarea | HU | Responsable | Estado | Notas |
|---|---|---|---|---|
| A1 Backend + Flyway + entidades | HU-01a | A | 🟨 | Código entregado en la Parte 1; falta compilar y arrancar |
| A2 Seeder usuarios | HU-01b | A | 🟨 | `SeederUsuarios` entregado; falta verificar conteos |
| A3 Seeder operación | HU-01c | A | 🟨 | `SeederOperacion` entregado; falta verificar conteos |
| A4 Errores y paginación | HU-02 | A | 🟨 | `common/` entregado; el 422 se prueba con HU-03 |
| A5 Login y JWT | HU-04 | A | ⬜ | Parte 2 |
| A6 Registro | HU-03 | A | ⬜ | Parte 2 |
| A7 Logout con revocación | HU-05 | A | ⬜ | |
| A8 Recuperar contraseña y correo | HU-06 | A | ⬜ | |
| A9 Seguridad por rol | HU-08 | A | ⬜ | |
| A10 Perfil backend | HU-07 | A | ⬜ | |
| B1 Esqueleto frontend | HU-07a | B | ⬜ | |
| B2 Login y sesión | HU-04 | B | ⬜ | Parte 2 |
| B3 Registro | HU-03 | B | ⬜ | Parte 2 |
| B4 Recuperar y restablecer | HU-06 | B | ⬜ | |
| B5 Rutas protegidas, navbar y perfil | HU-07/05 | B | ⬜ | |
| D Despliegue con HTTPS | HU-09 | A+B | ⬜ | |

## Estado por historia
| HU | Nombre | Estado | Notas |
|---|---|---|---|
| 00 | Entorno y repositorio | ⬜ | |
| 01 | Esqueleto backend y BD | 🟨 | Código entregado, pendiente de verificar |
| 02 | Errores globales y paginación | 🟨 | Código entregado, pendiente de verificar |
| 03 | Registro | ⬜ | |
| 04 | Login JWT | ⬜ | |
| 05 | Logout con revocación | ⬜ | |
| 06 | Recuperar contraseña | ⬜ | |
| 07 | Rutas protegidas, navbar y perfil | ⬜ | |
| 08 | Seguridad por rol | ⬜ | |
| 09 | Primer despliegue HTTPS | ⬜ | |
| 10 a 34 | Días 2, 3 y 4 | ⬜ | ver `03-HISTORIAS.md` |

## Archivos que ya existen
**Backend** (`backend/`): `pom.xml`, `.gitignore`, `src/main/resources/db/migration/V1__esquema.sql`, `application.properties.example`.
`src/main/java/com/rutaoax/`:
- `RutaoaxApplication`
- `common/`: `ApiError`, `PageResponse`, `Zona`, `ZonaCatalogo`, `BadRequestException`, `UnauthorizedException`, `ForbiddenException`, `NotFoundException`, `ConflictException`, `GlobalExceptionHandler`, `validation/PasswordSegura`, `validation/PasswordSeguraValidator`
- `config/`: `PasswordConfig`, `SecurityConfig` (**TEMPORAL, permite todo; HU-04 la reemplaza completa**), `SeederUsuarios`, `SeederOperacion`
- `usuario/`: `Usuario`, `Rol`, `RolNombre`, `UsuarioRepository`, `RolRepository`
- `negocio/`: `Negocio`, `Repartidor`, sus repositorios
- `vehiculo/`: `Vehiculo`, `TipoVehiculo`, `EstadoVehiculo`, `VehiculoRepository`
- `recorrido/`: `Recorrido`, `EstadoRecorrido`, `RecorridoRepository`
- `pedido/`: `Pedido`, `Modalidad`, `EstadoPedido`, `PedidoRepository`
- `mensaje/`: `Mensaje`, `MensajeRepository`
- `auth/`: `TokenRevocado`, `PasswordResetToken`, sus repositorios
- `notificacion/`: `NotificacionLog`, `Canal`, `EstadoEnvio`, `NotificacionLogRepository`
**Frontend:** _(vacío)_

## Bitácora de sesiones con IA
| Fecha | Tarea | Quién | IA / cuenta | Resultado | Archivos creados o modificados |
|---|---|---|---|---|---|
| ________ | Parte 1: HU-01 (a, b, c) y HU-02 | ________ | Claude (esta conversación) | Código entregado, sin compilar en el entorno de la IA; verificación pendiente | todo `backend/` listado arriba |

## Decisiones técnicas tomadas
Formato: fecha · decisión · motivo · documento corregido.
- Spring Boot **3.5.7** (no 4.x), Java 21: jjwt 0.12.x usa Jackson 2 y el código que generan las IA es de la línea 3.x. Corregido en `00-CONTEXTO.md`.
- Seeders separados: `SeederUsuarios` (@Order 1) y `SeederOperacion` (@Order 2), ambos idempotentes. Corregido en `00`, `03` y `08`.
- `SecurityConfig` provisional con `permitAll` durante la Parte 1; HU-04 la sustituye entera (mismo archivo, no se crea otra).
- Sin Lombok: getters y setters explícitos. Fechas `creadoEn`/`actualizadoEn` se asignan en `@PrePersist`/`@PreUpdate`.
- Coordenadas del seed: `ZonaCatalogo.coordenadas(zona, n)` con `n` consecutivo del 1 al 15 (determinista). Los pedidos nuevos usarán su id.
- `GlobalExceptionHandler` también cubre 405 (método no permitido) y parámetros mal formados como 400.
- El correo de bienvenida y `EmailService` se agregan en HU-06; `app.dev.log-reset-link` solo en local.
- `ddl-auto=none`; se verifican columnas y conteos con consultas SQL.

## Problemas abiertos
| # | Descripción | Archivo/HU | Cómo reproducir |
|---|---|---|---|
| 1 | El código de la Parte 1 no se compiló en el entorno donde se escribió (sin Maven ni red) | todo `backend/` | `mvn -q compile` y `mvn spring-boot:run` (ver `EJECUTAR-PARTE-1.md`) |

## Cómo probar lo que hay hoy
Backend: seguir `docs/dia-1/EJECUTAR-PARTE-1.md` (base local `rutaoax`, copiar `application.properties.example` a `application.properties`, `mvn spring-boot:run`).
Bruno: abrir `bruno/`, entorno `local`; hoy solo funciona `Error/Recurso inexistente 404`.
