# 00 · CONTEXTO DE RUTAOAX (pega este archivo en cada chat con una IA)

## Qué es
RutaOax es un sistema web de logística de entregas en Oaxaca de Juárez. Conecta negocios, repartidores (de empresa o independientes) y clientes.
Es el proyecto final del curso y debe cumplir la rúbrica de `p_final.docx` (ver `08-RUBRICA-Y-DEMO.md`). Plazo total: 4 días. Se desarrolla con IA en varias conversaciones, por eso **estos documentos son la única fuente de verdad**.

## Stack (no cambiar)
| Capa | Tecnología |
|---|---|
| Backend | Java 21, Spring Boot 3.5.7 (no usar 4.x: jjwt y el código que generan las IA esperan Jackson 2), Spring Web, Spring Security, Spring Data JPA, Bean Validation, Spring Mail, Maven |
| JWT | jjwt 0.12.x, HS256 |
| BD | MySQL 8 (NO MariaDB), Flyway, `ddl-auto=none` |
| Frontend | React + Vite, react-router-dom, axios, react-leaflet (Leaflet), CSS puro |
| Comunicación | Postfix + JavaMailSender (correo), Twilio SMS, Twilio WhatsApp |
| Pruebas API | Bruno (carpeta `bruno/` dentro del repo) |
| Servidor | VPS 179.197.84.34, Nginx, Certbot, systemd |
| URL final | https://rutaoax.devs8.tech · API: https://rutaoax.devs8.tech/api |

## Estructura del repositorio
```
rutaoax/
  backend/            Spring Boot (paquete base com.rutaoax)
  frontend/           React + Vite
  bruno/              colección Bruno
  docs/               estos documentos
  deploy/             rutaoax.service, nginx-rutaoax.conf, deploy.ps1
  README.md           documentación final de entrega (ER, credenciales, enlaces)
```

### Backend: un paquete por módulo (monolito modular)
```
com.rutaoax
  config/        SecurityConfig, PasswordConfig, SeederUsuarios y SeederOperacion (CommandLineRunner)
  common/        ApiError, PageResponse, GlobalExceptionHandler (@RestControllerAdvice), excepciones propias
  auth/          AuthController, AuthService, JwtService, JwtFilter, dto/
  usuario/       Usuario, Rol, UsuarioRepository, PerfilController, AdminUsuarioController
  negocio/       Negocio, Repartidor, EquipoController, ResumenController
  vehiculo/      Vehiculo, VehiculoController, VehiculoService
  pedido/        Pedido, PedidoController, PedidoService, DisponiblesController
  recorrido/     Recorrido, RecorridoController, RecorridoService
  mensaje/       Mensaje, MensajeController, MensajeService
  notificacion/  NotificacionService, EmailService, SmsService, WhatsAppService, NotificacionLog
```
Dentro de cada módulo: `Entidad`, `Repository`, `Service`, `Controller`, `dto/`.

### Frontend
```
src/
  api/          cliente axios (http.js) + un archivo por módulo (pedidosApi.js, ...)
  auth/         AuthContext, ProtectedRoute, RoleRoute
  components/   ui/ (Modal, ConfirmDialog, Pagination, Feedback, StatusBadge, Field), layout/ (Sidebar, Topbar, AppLayout)
  pages/        auth/ negocio/ repartidor/ independiente/ cliente/ admin/ perfil/
  styles/       estilos.css (viene del diseño ya hecho)
  utils/        geo.js (sortByNearest), format.js
```

## Reglas técnicas obligatorias
1. **API REST** con `@RestController` y `@RequestMapping("/api/...")`. Contrato exacto en `02-API.md`. No inventar endpoints, campos ni tablas. Si falta algo, proponerlo y esperar autorización.
2. **DTOs siempre.** Nunca devolver entidades JPA. Mapeo manual con métodos estáticos `from(entidad)` en los DTO (sin MapStruct).
3. **Validación solo con Bean Validation** (`@Valid`, `@NotBlank`, `@Size`, `@Email`, `@Pattern`...) en los DTOs de entrada. Prohibido validar a mano en controladores. Mensajes en español.
4. **Contraseña segura:** mínimo 8 caracteres, una mayúscula, un número y un carácter especial. Regex: `^(?=.*[A-Z])(?=.*\d)(?=.*[^A-Za-z0-9]).{8,}$` (anotación propia `@PasswordSegura` en backend; misma regex en el frontend, mensaje debajo del input).
5. **BCrypt** para contraseñas. Nunca texto plano.
6. **JPA:** relaciones `LAZY`; los listados usan `JOIN FETCH` o `@EntityGraph` para evitar N+1. `spring.jpa.open-in-view=false`.
7. **Paginación y filtros del lado del servidor** con `Pageable` (`page` desde 0, `size`). El frontend nunca carga todo para filtrar en el cliente.
8. **Errores JSON** con `@RestControllerAdvice` (formato en `02-API.md`): 400 JSON mal formado, 401 sin token o inválido, 403 sin permiso, 404 no existe, 409 conflicto de negocio, 422 validación de campos, 500 inesperado.
9. **Autorización:** `SecurityFilterChain` por rutas + `@PreAuthorize("hasRole('...')")` en servicios o controladores. Además, **cada servicio comprueba pertenencia** (un negocio solo toca lo suyo, un cliente solo ve lo suyo).
10. **Frontend:** nunca `alert()` ni `confirm()`. Errores de validación bajo cada input (en tiempo real y al enviar). Modal de confirmación en acciones destructivas. Estados de carga y de error de red visibles en toda pantalla que llama a la API. Responsivo (móvil y escritorio).
11. **Secretos fuera del repo.** `application.properties` está en `.gitignore`; se versiona solo `application.properties.example`.
12. **Notificaciones nunca rompen la operación principal.** Si falla un correo/SMS/WhatsApp, se registra en `notificaciones_log` con estado FALLIDO y la operación continúa. Nunca simular un envío como real.
13. **Coordenadas:** no hay geocodificación externa. El backend calcula `lat/lng` del pedido a partir de la `zona` (tabla en `01-BASE-DATOS.md`) más un desplazamiento pequeño determinista (por id).
14. **No cambiar la arquitectura, nombres de rutas ni contratos** sin autorización. No tocar módulos que no sean de la tarea.

## Roles
`ADMIN`, `NEGOCIO`, `REPARTIDOR`, `CLIENTE`. Se guardan en `roles` y se relacionan con `usuarios` por `usuario_roles` (N:M).
- **Repartidor de empresa:** rol REPARTIDOR con `repartidores.negocio_id` distinto de null.
- **Repartidor independiente:** rol REPARTIDOR con `negocio_id` null.
- El frontend decide qué diseño mostrar leyendo `usuario.perfil.tipoRepartidor` (`EMPRESA` o `INDEPENDIENTE`).

### Qué puede hacer cada rol
| Recurso | ADMIN | NEGOCIO | REPARTIDOR empresa | REPARTIDOR independiente | CLIENTE |
|---|---|---|---|---|---|
| Usuarios (CRUD) | todos | solo su equipo | no | no | no |
| Vehículos | todos | los suyos | no | no | no |
| Pedidos | todos | los suyos | solo los de su recorrido | disponibles + los que tomó | los suyos |
| Asignar pedidos | sí | sí (a su equipo) | no | se autoasigna (tomar) | no |
| Cambiar estado de entrega | sí | no | sus paradas | sus paradas | no |
| Mensajes | no | los de sus pedidos | no | los de sus entregas | los de sus pedidos |
| Seguimiento | sí | sí | no | no | sí |

## Estados del pedido
`PENDIENTE → ASIGNADO → EN_CAMINO → ENTREGADO | NO_ENTREGADO`
- `PENDIENTE → ASIGNADO`: el negocio asigna o el independiente toma.
- `ASIGNADO → EN_CAMINO`: el repartidor inicia el recorrido (todas sus paradas ASIGNADO pasan a EN_CAMINO).
- `EN_CAMINO → ENTREGADO | NO_ENTREGADO`: lo marca el repartidor en la parada.
- `ASIGNADO → PENDIENTE`: soltar (independiente) o desasignar (negocio), solo antes de EN_CAMINO.
- `NO_ENTREGADO → PENDIENTE`: el negocio lo reasigna.
Cualquier otra transición responde **409**.

## Modalidad del pedido
- `NEGOCIO`: lo crea un negocio y lo asigna a su equipo.
- `ABIERTO`: lo ve cualquier repartidor independiente en "Entregas disponibles". Lo crea un cliente (envío) o lo publica un negocio. Un cliente solo puede crear pedidos `ABIERTO` (con `negocio_id` null).

## A quién llega un mensaje (decide el backend)
- Cliente escribe sobre un pedido: si el pedido tiene recorrido y su repartidor es de empresa → va al **propietario del negocio**; si el repartidor es independiente → va **directo al repartidor**; si no hay recorrido pero hay negocio → al propietario del negocio; si es ABIERTO sin repartidor → **409** ("Aún no hay quién atienda este pedido").
- Negocio o repartidor independiente responde → va al **cliente** del pedido (si el pedido no tiene `cliente_id` → 409).
- Cada hilo está ligado a un `pedido_id` y la interfaz siempre muestra "Pedido #id".

## Notificaciones (acciones reales del sistema)
| Canal | Evento | Destino |
|---|---|---|
| Correo | Recuperar contraseña; registro de cuenta (bienvenida) | email del usuario |
| SMS | El pedido pasa a EN_CAMINO | `cliente_telefono` |
| WhatsApp | El pedido pasa a ENTREGADO | `cliente_telefono` |
Todo envío (éxito o fallo) se guarda en `notificaciones_log`.

## Diseño visual
Ya existe un diseño HTML/CSS por pantalla (carpeta `diseno/`, archivo `css/estilos.css`). El frontend debe **reproducir esos diseños en React**, reutilizando las clases de `estilos.css`. Paleta: beige de fondo, blanco en superficies, azul para negocio/empresa/cliente, morado para el independiente, verde/ámbar/rojo solo para estados.

| Ruta React | Diseño |
|---|---|
| `/login`, `/registro`, `/recuperar`, `/restablecer` | `index.html` (adaptado) |
| `/negocio` | `negocio-panel.html` |
| `/negocio/pedidos` | `negocio-pedidos.html` |
| `/negocio/equipo` | `negocio-equipo.html` |
| `/negocio/vehiculos` | `negocio-vehiculos.html` |
| `/negocio/mensajes` | `negocio-mensajes.html` |
| `/repartidor` | `repartidor-ruta.html` |
| `/repartidor/parada/:pedidoId` | `repartidor-parada.html` |
| `/repartidor/historial` | `repartidor-historial.html` |
| `/independiente` | `independiente-disponibles.html` |
| `/independiente/ruta` | `independiente-ruta.html` |
| `/independiente/mensajes` | `independiente-mensajes.html` |
| `/cliente` | `cliente-pedido.html` |
| `/cliente/mensajes` | `cliente-mensajes.html` |
| `/admin/*`, `/perfil` | tablas y formularios con los mismos componentes (sin diseño propio) |

Redirección tras login: ADMIN → `/admin/usuarios`; NEGOCIO → `/negocio`; REPARTIDOR → `/repartidor` (empresa) o `/independiente`; CLIENTE → `/cliente`.

## Convenciones
- Idioma de la interfaz y mensajes: español de México. Nombres de código: tablas, columnas, endpoints y campos JSON en español (`pedidos`, `clienteNombre`), igual que en este contrato.
- JSON en camelCase; columnas SQL en snake_case.
- Fechas: `fecha` en `YYYY-MM-DD`; fecha y hora en ISO-8601.
- Commits: `tipo(módulo): descripción` (ej. `feat(pedidos): asignar pedidos a repartidor`).
- Puerto del backend: 9250. En desarrollo el frontend usa `VITE_API_URL=http://localhost:9250/api`; en producción `/api`.

## Cómo se prueba cada cosa
Cada tarea termina con: (1) `mvn -q compile` o `npm run build` sin errores, (2) una petición de Bruno que lo demuestre, (3) revisar la pantalla en el navegador, (4) commit. Las tareas se definen en `03-HISTORIAS.md`.
