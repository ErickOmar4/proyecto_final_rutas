# EJECUTAR EL DÍA 2 · PARTE 1 · HU-10 (vehículos) y HU-11 (equipo de reparto)

**Meta:** un negocio administra su flota y su equipo desde pantallas reales conectadas al backend, con filtros y paginación del servidor, sin ver nada de otros negocios.
**Tiempo:** 45 a 60 minutos. Comandos en PowerShell.

> Como en las partes anteriores, este código se escribió sin poder compilarlo ni ejecutarlo en el entorno donde se generó. Los pasos 2 y 4 son la primera prueba real; el paso 8 dice qué hacer si algo falla.

## 1. Copiar el zip al repositorio
Copia el contenido de `rutaoax-dia2p1` sobre tu repo (elige *Reemplazar*). Esto **reemplaza** archivos que ya tenías, es lo esperado:
- `backend/`: `VehiculoRepository`, `RecorridoRepository`, `PedidoRepository` y `RepartidorRepository` (ahora con consultas nuevas), y agrega `NegocioContexto`, `EquipoController`, `EquipoService`, `VehiculoController`, `VehiculoService` y sus DTO.
- `frontend/`: `App.jsx`, `styles/estilos.css` y `utils/format.js` (reemplazados) y archivos nuevos en `api/`, `components/ui/`, `utils/` y `pages/negocio/`.
- `bruno/`: carpetas nuevas `Vehiculos` y `Equipo`, y `Auth/Login negocio 2`.
- `docs/`: `02-API.md`, `06-ESTADO.md` y `dia-1/01-INTERFACES-COMPARTIDAS.md` actualizados.

## 2. Compilar y arrancar el backend
```powershell
cd backend
mvn -q compile
mvn spring-boot:run
```
`compile` sin errores y `Started RutaoaxApplication`.
Importante: Spring valida las consultas `@Query` **al arrancar**. Si alguna estuviera mal escrita, el arranque falla con `Validation failed for query for method ...`; en ese caso ve al paso 8.

## 3. Probar el backend con Bruno (entorno `local`)
Ejecuta **toda** la colección. Debe quedar todo en verde:
| Carpeta | Esperado |
|---|---|
| Auth (1 a 18), Error, Perfil, Seguridad | igual que antes (ahora `Auth/Login negocio 2` también) |
| Vehiculos 1 a 13 | listar y filtrar (200), crear (201), placa repetida (409), datos inválidos (422), editar (200), de otro negocio (404), admin ve todos (200), disponibles (200), admin no crea (403), eliminar (204), eliminado (404) |
| Equipo 1 a 12 | listar y buscar (200), crear (201), correo repetido (409), inválido (422), editar (200), disponibles (200), de otro negocio (404), el nuevo inicia sesión (200), quitar (204), quitado no inicia sesión (401), admin (403) |
Si ya creaste vehículos o repartidores a mano, las pruebas siguen pasando (usan "mayor o igual").

## 4. Verificar en MySQL
```sql
SELECT placa, tipo, capacidad, estado FROM vehiculos ORDER BY id;     -- placas en MAYÚSCULAS
SELECT email, activo FROM usuarios WHERE email LIKE 'miembro%';        -- activo = 0 (los que creó Bruno y quitó)
```

## 5. Probar las pantallas
Con el backend corriendo, `cd frontend; npm run dev`. Entra con `negocio1@rutaoax.com` / `Ruta#2026`.

**Vehículos (`/negocio/vehiculos`)**
| Prueba | Esperado |
|---|---|
| Abrir la pantalla | 6 tarjetas (los vehículos de Dulces del Valle), sin ventanas emergentes |
| Tarjeta OAX-214 y OAX-880 | etiqueta "En uso hoy" y "Hoy: Luis Martínez" / "Hoy: Ana Ruiz" **si el seed corrió hoy** (los recorridos de ejemplo llevan la fecha del día que se cargó la base) |
| OAX-880 | aviso ámbar "El seguro vence en N días" (N = días que faltan de los 5 iniciales) |
| OAX-502 (Combi) | etiqueta "En mantenimiento" |
| Buscar `214` | tras una pausa corta aparece solo OAX-214 |
| Filtrar por tipo "Motocicleta" y por estado "En mantenimiento" | la lista cambia; sin resultados muestra "Ningún vehículo coincide con los filtros." |
| "+ Registrar vehículo" con todo vacío y Guardar | errores rojos bajo cada campo, sin guardar |
| Capacidad `0` | "La capacidad debe ser al menos 1" |
| Placa `oax-214` (ya existe) | mensaje rojo "Ya existe un vehículo con esa placa" |
| Placa nueva `TEST-01`, capacidad 5 | mensaje verde y aparece como `TEST-01` (mayúsculas); ahora hay 7 vehículos y **2 páginas** |
| Paginación | "Página 1 de 2", "Siguiente" lleva a la segunda página con 1 vehículo |
| Editar `TEST-01` → capacidad 8 | mensaje verde y se actualiza |
| Eliminar `TEST-01` | cuadro de confirmación, y al aceptar mensaje verde; vuelve a 6 vehículos |
| Eliminar OAX-214 **el mismo día del seed** | mensaje rojo "El vehículo tiene recorridos programados..." (no se elimina) |
| Cerrar sesión y entrar como `negocio2@rutaoax.com` | solo ve OAX-120, OAX-455, BIC-11 y OAX-908 |

**Equipo (`/negocio/equipo`)** con `negocio1@`
| Prueba | Esperado |
|---|---|
| Abrir la pantalla | tabla con Ana Ruiz, Jorge Pérez y Luis Martínez (orden alfabético) |
| Luis Martínez | "Motocicleta OAX-214", "0 de 2", etiqueta "En ruta" (con datos del día del seed) |
| Ana Ruiz | "Camioneta OAX-880", "0 de 2", etiqueta "Con ruta asignada" |
| Jorge Pérez | "Sin vehículo", "—", etiqueta "Libre" |
| Buscar `luis` | solo Luis |
| "+ Agregar repartidor" vacío y Guardar | errores bajo cada campo; la contraseña muestra su lista de requisitos |
| Correo `repartidor1@rutaoax.com` | mensaje rojo "Ya existe una cuenta con ese correo" |
| Crear `Carlos Prueba`, `carlos@rutaoax.com`, `Equipo#2026` | mensaje verde con el aviso de compartir el correo y la contraseña; aparece en la tabla como "Libre" |
| Abrir otra ventana privada, entrar con `carlos@rutaoax.com` / `Equipo#2026` | entra a `/repartidor` con "Repartidor · Dulces del Valle" |
| Editar a Carlos (nombre y teléfono) | mensaje verde; el correo está bloqueado |
| Quitar a Carlos | confirmación y mensaje verde; desaparece de la tabla; en la ventana privada ya no puede volver a iniciar sesión |
| Quitar a Luis o a Ana **el mismo día del seed** | mensaje rojo "Tiene recorridos pendientes..." |
| Como `negocio2@` | solo aparece Rosa Cruz |
Con F12 → Consola no deben salir errores en rojo. Prueba también en ancho móvil (F12 → icono de dispositivo): los filtros y la tabla deben poder usarse (la tabla se desplaza hacia el lado).

## 6. Cerrar la parte
```powershell
cd ..    # raíz del repo
git add backend bruno
git commit -m "feat(vehiculos,equipo): crud de vehiculos y gestion del equipo de reparto"
git add frontend docs
git commit -m "feat(frontend): pantallas de vehiculos y equipo con filtros y paginacion"
git push
```
Luego:
1. **Tablero:** HU-10 y HU-11 → Done si todo pasó.
2. **`docs/06-ESTADO.md`:** cambia 🟨 por ✅ en HU-10, HU-11 y las filas A11, B6, A12 y B7; borra el problema abierto #1; llena fecha y "Quién". Si ya habías puesto repo, tablero y Figma en tu copia, pásalos al archivo nuevo.
3. **Recordatorio de despliegue:** el primer despliegue con HTTPS (HU-09) sigue pendiente. Si ya tienes el DNS listo, hazlo ahora (Parte B de `dia-1/EJECUTAR-PARTE-4.md`) y publica estos cambios con `./deploy/deploy.ps1`; entre más tarde lo hagas, más cosas tendrás que depurar juntas.

## 7. Qué sigue (Día 2 · Parte 2)
HU-12 (pedidos del negocio: tabla con paginación y filtros, crear, editar y borrar) y HU-13 (asignar pedidos a un repartidor con vehículo, con cupo y reglas 409). Es la parte más pesada del día: reutiliza `NegocioContexto`, el patrón de listados, `Pagination` y las listas de "disponibles" que acabas de crear.
Si sigues en esta conversación no necesito nada; en una nueva, súbele `00-CONTEXTO.md`, `02-API.md`, `03-HISTORIAS.md`, `06-ESTADO.md` y `dia-1/01-INTERFACES-COMPARTIDAS.md` (con todos los anexos).

## 8. Si algo falla
| Síntoma | Qué hacer |
|---|---|
| `mvn -q compile` marca un error | Copia el error completo y el archivo; pide a una IA (con `00-CONTEXTO.md` y los anexos de interfaces) el archivo completo corregido |
| Arranque: `Validation failed for query for method ...` | Una consulta `@Query` tiene un error de sintaxis: copia el mensaje completo y el repositorio indicado (`VehiculoRepository`, `RecorridoRepository`, `PedidoRepository` o `RepartidorRepository`) |
| Arranque: `No property ... found for type ...` | Un método de repositorio no coincide con el nombre del campo de la entidad: pega el mensaje y la entidad |
| Un listado responde 500 | Mira la consola del backend (el primer error en rojo) y pásalo con el servicio indicado |
| Bruno `Vehiculos` 4 da 422 | La placa generada no cumple el patrón: vuelve a ejecutar la petición |
| Bruno `Vehiculos 9` falla | El administrador debe ver al menos 10 vehículos: revisa `SELECT COUNT(*) FROM vehiculos;` |
| La pantalla muestra siempre "Cargando..." | El backend no está corriendo o hay un error de red/CORS: mira la consola del navegador (F12) |
| Los vehículos no dicen "En uso hoy" | El seed se cargó otro día: los recorridos de ejemplo tienen esa fecha. Es normal; se vuelve a ver al asignar pedidos (HU-13) |
| Al buscar, la lista parpadea | Es normal: espera 0.4 segundos tras dejar de escribir y consulta al servidor |
| Pantalla en blanco | F12 → Consola: copia el primer error rojo y el archivo que menciona |
Para pedir ayuda a una IA usa siempre: prompt maestro + `00-CONTEXTO.md` + el error completo + solo el archivo que falla.
