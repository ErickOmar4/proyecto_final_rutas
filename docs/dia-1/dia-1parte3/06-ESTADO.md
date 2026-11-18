# 06 · ESTADO ACTUAL (actualizar al cerrar cada tarea y cada día)

Última actualización: ________ · Día: 1 · Parte 3 entregada (HU-05, HU-06 y HU-08) · Quién: ________

## Resumen
Fase actual: **DÍA 1 · Parte 3 entregada** (HU-05, HU-06 y HU-08, **por verificar** con `docs/dia-1/EJECUTAR-PARTE-3.md`). Quedan HU-07 (perfil: backend y pantalla) y HU-09 (primer despliegue con HTTPS) para la Parte 4.
- Verificado por el equipo: backend arranca con la seguridad de la Parte 2 (login, registro y `/auth/me`). Se corrigió `SecurityConfig` (había dos beans `CorsConfigurationSource`).
- Parte 3 entrega: logout con revocación del token, recuperación de contraseña con servicio de correo (`EmailService`), correo de bienvenida al registrarse, seguridad por rol en el backend y las pantallas `/recuperar` y `/restablecer`.
- El correo real depende de Postfix (HU-26): hasta entonces los envíos quedan como FALLIDO en `notificaciones_log`, nunca simulados.
- Siguen sin existir: perfil (HU-07 backend y pantalla) y el despliegue (HU-09).
- Interfaces congeladas en `docs/dia-1/01-INTERFACES-COMPARTIDAS.md` (con anexos de las Partes 2 y 3).

## URLs y accesos
- Sitio: https://rutaoax.devs8.tech · API: https://rutaoax.devs8.tech/api · VPS: 179.197.84.34 · Backend local: http://localhost:9250/api · Frontend local: http://localhost:5173
- Repo: ________ · Tablero: ________ · Figma: ________
- Credenciales demo: admin `admin@rutaoax.com` / `Admin#2026`; resto `Ruta#2026` (negocio1@, negocio2@, repartidor1@ a repartidor4@, independiente1@, independiente2@, cliente1@ a cliente4@ … @rutaoax.com).
- MySQL local: base `rutaoax`, usuario `rutaoax_app`.

## Plan del Día 1 (tareas de `docs/dia-1/02-TAREAS-DIA-1.md`)
Estados: ⬜ pendiente · 🟨 en curso o por verificar · ✅ hecha y verificada · ❌ bloqueada
| Tarea | HU | Responsable | Estado | Notas |
|---|---|---|---|---|
| A1 Backend + Flyway + entidades | HU-01a | A | ✅ | Verificado por el equipo |
| A2 Seeder usuarios | HU-01b | A | ✅ | Verificado por el equipo |
| A3 Seeder operación | HU-01c | A | ✅ | Verificado por el equipo |
| A4 Errores y paginación | HU-02 | A | ✅ | Verificado por el equipo |
| A5 Login y JWT | HU-04 | A | ✅ | Verificado (arranca y funciona tras corregir CORS) |
| A6 Registro | HU-03 | A | ✅ | Verificado |
| A7 Logout con revocación | HU-05 | A | 🟨 | Entregado en la Parte 3; falta verificar |
| A8 Recuperar contraseña y correo | HU-06 | A | 🟨 | Entregado en la Parte 3; el envío real queda FALLIDO hasta tener Postfix |
| A9 Seguridad por rol | HU-08 | A | 🟨 | Entregado en la Parte 3; falta verificar |
| A10 Perfil backend | HU-07 | A | ⬜ | |
| B1 Esqueleto frontend | HU-07a | B | ✅ | Verificado |
| B2 Login y sesión | HU-04 | B | ✅ | Verificado |
| B3 Registro | HU-03 | B | ✅ | Verificado |
| B4 Recuperar y restablecer | HU-06 | B | 🟨 | Entregado en la Parte 3; falta verificar |
| B5 Rutas protegidas, navbar y perfil | HU-07/05 | B | 🟨 | ADELANTADO: ProtectedRoute, RoleRoute, NoAutorizado y menú de usuario con cerrar sesión. Falta la pantalla de perfil |
| D Despliegue con HTTPS | HU-09 | A+B | ⬜ | |

## Estado por historia
| HU | Nombre | Estado | Notas |
|---|---|---|---|
| 00 | Entorno y repositorio | ⬜ | |
| 01 | Esqueleto backend y BD | ✅ | Verificado. V1 ajustado (ver Decisiones) |
| 02 | Errores globales y paginación | ✅ | Verificado |
| 03 | Registro | ✅ | Verificado |
| 04 | Login JWT | ✅ | Verificado |
| 05 | Logout con revocación | 🟨 | Backend entregado; el botón de cerrar sesión ya lo usa; falta verificar |
| 06 | Recuperar contraseña | 🟨 | Backend y pantallas entregados; correo real pendiente de Postfix |
| 07 | Rutas protegidas, navbar y perfil | 🟨 | Parcial (adelantado en la Parte 2); faltan perfil backend y pantalla |
| 08 | Seguridad por rol | 🟨 | Entregado; falta verificar con la carpeta Seguridad de Bruno |
| 09 | Primer despliegue HTTPS | ⬜ | |
| 10 a 34 | Días 2, 3 y 4 | ⬜ | ver `03-HISTORIAS.md` |

## Archivos que ya existen
**Backend** (`backend/`): `pom.xml`, `.gitignore`, `src/main/resources/db/migration/V1__esquema.sql`, `application.properties.example`.
`src/main/java/com/rutaoax/`:
- `RutaoaxApplication`
- `common/`: `ApiError`, `PageResponse`, `Zona`, `ZonaCatalogo`, `BadRequestException`, `UnauthorizedException`, `ForbiddenException`, `NotFoundException`, `ConflictException`, `GlobalExceptionHandler`, `validation/PasswordSegura`, `validation/PasswordSeguraValidator`
- `config/`: `PasswordConfig`, `SecurityConfig` (**con roles desde la Parte 3**), `AsyncConfig`, `SchedulingConfig`, `ApiErrorWriter`, `RestAuthenticationEntryPoint`, `RestAccessDeniedHandler`, `SeederUsuarios`, `SeederOperacion`
- `usuario/`: `Usuario`, `Rol`, `RolNombre`, `UsuarioRepository`, `RolRepository`, `UsuarioService`, `dto/` (`UsuarioDTO`, `PerfilDTO`)
- `negocio/`: `Negocio`, `Repartidor`, sus repositorios
- `vehiculo/`: `Vehiculo`, `TipoVehiculo`, `EstadoVehiculo`, `VehiculoRepository`
- `recorrido/`: `Recorrido`, `EstadoRecorrido`, `RecorridoRepository`
- `pedido/`: `Pedido`, `Modalidad`, `EstadoPedido`, `PedidoRepository`
- `mensaje/`: `Mensaje`, `MensajeRepository`
- `auth/`: `TokenRevocado`, `PasswordResetToken`, sus repositorios, `JwtService`, `JwtFilter`, `UsuarioPrincipal`, `AuthService`, `AuthController`, `PasswordResetService`, `TokenCleanupJob`, `dto/` (`LoginRequest`, `LoginResponse`, `RegisterRequest`, `TipoCuenta`, `ForgotPasswordRequest`, `ResetPasswordRequest`)
- `notificacion/`: `NotificacionLog`, `Canal`, `EstadoEnvio`, `NotificacionLogRepository`, `EmailService`
**Endpoints vivos:** `POST /api/auth/login`, `/register`, `/logout`, `/forgot-password`, `/reset-password` y `GET /api/auth/me`.
**Frontend** (`frontend/`): `package.json`, `vite.config.js`, `index.html`, `.env.development`, `.env.production`; `src/`: `main.jsx`, `App.jsx`, `styles/estilos.css`, `api/` (`http`, `errores`, `authApi`), `auth/` (`AuthContext`, `ProtectedRoute`, `RoleRoute`, `RedirectInicio`, `rutas`), `components/ui/` (`Field`, `Feedback`, `Modal`, `ConfirmDialog`, `StatusBadge`), `components/layout/` (`AppLayout`, `Sidebar`, `Topbar`, `menus`), `pages/` (`PaginaPendiente`, `NoAutorizado`, `auth/` `AuthShell`, `LoginPage`, `RegistroPage`, `RecuperarPage` y `RestablecerPage` ya completas), `utils/` (`validaciones`, `format`).

## Bitácora de sesiones con IA
| Fecha | Tarea | Quién | IA / cuenta | Resultado | Archivos creados o modificados |
|---|---|---|---|---|---|
| ________ | Parte 1: HU-01 (a, b, c) y HU-02 | ________ | Claude (esta conversación) | Verificado por el equipo; se corrigió una línea del SQL | `backend/` base |
| ________ | Parte 2: HU-03, HU-04 (backend y frontend) | ________ | Claude (esta conversación) | Verificado por el equipo; se corrigió `SecurityConfig` (CORS duplicado) | ver lista de archivos |
| ________ | Parte 3: HU-05, HU-06, HU-08 | ________ | Claude (esta conversación) | Entregado sin compilar ni ejecutar en el entorno de la IA; verificación pendiente | ver lista de archivos |

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
- **V1 corregido por el equipo:** `fk_ped_negocio` quedó sin `ON DELETE SET NULL` (MySQL no permite `CHECK` sobre una columna con acción referencial, error 3823). Se conserva `ck_ped_dueno`. Consecuencia: no se puede borrar un negocio con pedidos (manejar 409 en el Día 4). Corregido en `01-BASE-DATOS.md`.
- JWT: HS256, claims `sub`, `uid`, `roles`, `jti`, `iat`, `exp`; el filtro usa el usuario y los roles de la base en cada petición (no confía solo en los roles del token).
- `JwtFilter` no es `@Component`: lo crea `SecurityConfig` (evita que se ejecute dos veces).
- Los métodos `@AssertTrue` llamados `isXxxValido()` reportan el error en el campo `xxx` (convención en `GlobalExceptionHandler`).
- Frontend: React 18, React Router 6, axios, Vite 5; token en `sessionStorage` (`rutaoax_token`). `ProtectedRoute`, `RoleRoute` y el cierre de sesión se adelantaron de HU-07 porque el layout los necesita.
- `SecurityConfig`: `.cors(Customizer.withDefaults())` en lugar de inyectar `CorsConfigurationSource` (Spring tiene otro bean de ese tipo y fallaba el arranque). Spring toma el bean llamado `corsConfigurationSource`.
- HU-05/06: la recuperación vive en `PasswordResetService` (no en `AuthService`); `AuthService` agrega `logout` y el correo de bienvenida. Token de recuperación: 32 bytes aleatorios, se guarda solo el hash SHA-256 (hex), vence en 30 minutos y es de un solo uso; al pedir uno nuevo se invalidan los anteriores.
- `EmailService.enviar` es `@Async` y nunca lanza excepciones: registra ENVIADO o FALLIDO en `notificaciones_log`. Eventos: `REGISTRO` y `RECUPERACION`.
- HU-08: matriz de roles en `SecurityConfig` + `@EnableMethodSecurity`. El filtro solo comprueba el rol; las reglas de pertenencia y de tipo de repartidor (empresa o independiente) se aplican en cada servicio desde el Día 2.
- El equipo usa JDK 23 en local: no hay problema porque el proyecto compila con `java.version=21`; el VPS debe usar JRE 21.

## Problemas abiertos
| # | Descripción | Archivo/HU | Cómo reproducir |
|---|---|---|---|
| 1 | El código de la Parte 3 no se compiló ni ejecutó en el entorno donde se escribió | `backend/` y `frontend/` | Seguir `docs/dia-1/EJECUTAR-PARTE-3.md` |
| 3 | Correo real: Postfix, SPF y DKIM aún no están configurados; los envíos quedan FALLIDO | HU-26 (Día 3) | `SELECT estado FROM notificaciones_log;` |
| 2 | Borrar un negocio con pedidos fallará por la llave foránea; falta responder 409 legible | Día 4 (admin) | `DELETE /admin/usuarios/{id}` de un negocio con pedidos |

## Cómo probar lo que hay hoy
Backend: seguir `docs/dia-1/EJECUTAR-PARTE-1.md` (base local `rutaoax`, copiar `application.properties.example` a `application.properties`, `mvn spring-boot:run`).
Frontend: `cd frontend && npm install && npm run dev` → http://localhost:5173 (con el backend corriendo).
Bruno: abrir `bruno/`, entorno `local`; ver en `EJECUTAR-PARTE-2.md` qué peticiones deben estar en verde hoy.
