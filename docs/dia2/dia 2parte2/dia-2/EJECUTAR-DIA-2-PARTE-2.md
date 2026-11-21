# EJECUTAR EL DÍA 2 · PARTE 2 · HU-12 (pedidos) y HU-13 (asignación)

**Meta:** el negocio crea y administra sus pedidos con filtros y paginación del servidor, y asigna varios a un repartidor con vehículo y fecha, respetando cupo y reglas.
**Tiempo:** 60 a 75 minutos. Comandos en PowerShell.

> Como en las partes anteriores, este código se escribió sin poder compilarlo ni ejecutarlo en el entorno donde se generó. Los pasos 2 y 3 son la primera prueba real; el paso 8 dice qué hacer si algo falla.

## 1. Copiar el zip al repositorio
Copia el contenido de `rutaoax-dia2p2` sobre tu repo (elige *Reemplazar*). Reemplaza archivos que ya tenías, es lo esperado:
- `backend/`: **reemplaza** `PedidoRepository` y `RecorridoRepository` (ahora con más consultas); agrega `PedidoController`, `PedidoService`, `AsignacionService`, `RecorridoService`, `Geo`, `ZonaController` y los DTO de `pedido/` y `recorrido/`.
- `frontend/`: **reemplaza** `App.jsx`, `styles/estilos.css` y `utils/format.js`; agrega `api/pedidosApi.js`, `api/zonasApi.js` y `pages/negocio/` (`PedidosPage`, `PedidoFormModal`, `AsignarPanel`).
- `bruno/`: carpeta nueva `Pedidos`.
- `docs/`: `02-API.md`, `06-ESTADO.md`, `08-RUBRICA-Y-DEMO.md` y `dia-1/01-INTERFACES-COMPARTIDAS.md` actualizados.

## 2. Compilar y arrancar el backend
```powershell
cd backend
mvn -q compile
mvn spring-boot:run
```
`compile` sin errores y `Started RutaoaxApplication`. Si el arranque falla con `Validation failed for query for method ...`, una consulta `@Query` tiene un error: ve al paso 8.

## 3. Probar el backend con Bruno (entorno `local`)
Ejecuta **toda** la colección: debe quedar en verde.
| Carpeta | Esperado |
|---|---|
| Auth, Equipo, Error, Perfil, Seguridad, Vehiculos | igual que antes |
| Pedidos 1 a 6 | listar, filtrar por estado, buscar por cliente, zonas (10), crear pedidos A y B (201) |
| Pedidos 7 a 13 | datos inválidos (422), ver, editar, de otro negocio (404), cliente no puede crear (403), admin ve todos, cliente ve solo los suyos |
| Pedidos 14 a 20 | repartidor libre ese día, vehículo con cupo 1, **excede capacidad (409)**, **asignar (200)**, ya asignado (409), repartidor de otro negocio (404), fecha pasada (422) |
| Pedidos 21 a 26 | desasignar (200), desasignar otra vez (409), eliminar A y B (204), eliminado (404), eliminar el vehículo (204) |
La última petición demuestra otra regla: al desasignar el único pedido, el recorrido vacío se elimina; si no fuera así, el vehículo seguiría "programado" y no se podría borrar.

## 4. Verificar en MySQL
```sql
SELECT COUNT(*) FROM recorridos;                     -- 6 (los de ejemplo): Bruno no deja recorridos sueltos
SELECT estado, COUNT(*) FROM pedidos GROUP BY estado; -- los 15 de ejemplo siguen igual (Bruno borró los suyos)
SELECT id, lat, lng, zona FROM pedidos ORDER BY id DESC LIMIT 3;
```

## 5. Probar las pantallas
`cd frontend; npm run dev` y entra con `negocio1@rutaoax.com` / `Ruta#2026` → **Pedidos y asignación**.
**Datos de ejemplo de este negocio:** 11 pedidos → 4 sin asignar (2 de ellos "Publicado"), 2 asignados (Ana Ruiz), 2 en camino (Luis Martínez), 2 entregados y 1 no entregado.

**Listado y filtros**
| Prueba | Esperado |
|---|---|
| Abrir la pantalla | tabla con los pedidos más recientes primero; con 10 por página hay **2 páginas** |
| Cambiar "Por página" a 20 | una sola página con los 11 |
| Estado "Sin asignar" | 4 pedidos |
| Tipo "Publicados" | los 2 que dicen "· Publicado" |
| Zona "Jalatlaco" | solo los de esa zona |
| Buscar `elena` | los pedidos de Elena Gómez |
| Buscar `#8` o `8` | aparece el pedido #8 |
| Combinaciones sin resultados | "Ningún pedido coincide con los filtros." |
| Casillas | solo se pueden marcar los "Sin asignar"; la casilla del encabezado marca todos los de la página |
| Pedidos "En camino" o "Entregado" | no muestran botones Editar ni Eliminar |

**Crear, editar y eliminar**
| Prueba | Esperado |
|---|---|
| "+ Nuevo pedido" y Guardar vacío | errores rojos bajo cada campo, sin ventanas emergentes |
| Total `-5`, artículos `0`, teléfono `123` | un error bajo cada campo |
| Crear un pedido válido (zona Centro Histórico) | mensaje verde y aparece arriba como "Sin asignar" |
| Escribir el correo `cliente1@rutaoax.com` al crearlo | queda enlazado: al entrar como `cliente1@` (Parte del Día 3) lo verá; por ahora solo comprueba que se guarda |
| Marcar "Publicar para repartidores independientes" y llenar el pago | queda con "· Publicado" |
| Editar un pedido sin asignar | mensaje verde y se actualiza |
| Eliminar un pedido sin asignar | confirmación y mensaje verde |

**Asignación (panel de la derecha)**
Antes de asignar, el panel dice "Asignar 0 pedidos" y el botón está desactivado.
| Prueba | Esperado |
|---|---|
| Marcar 2 pedidos sin asignar | "Asignar 2 pedidos" |
| Fecha de hoy | repartidores libres: **Jorge Pérez** (si el seed corrió hoy; si no, también Luis y Ana); vehículos libres: BIC-03, OAX-301 y OAX-777 (el Combi OAX-502 está en mantenimiento y no aparece) |
| Elegir Jorge Pérez + Automóvil OAX-777 | recuadro "Cupo del vehículo 15 pedidos / Seleccionados 2" |
| Elegir la bicicleta BIC-03 (cupo 4) y marcar 5 pedidos | aviso ámbar "Seleccionaste más pedidos de los que caben" y **Asignar desactivado** (crea pedidos de prueba si hace falta) |
| Asignar (2 pedidos, Jorge, OAX-777, hoy) | mensaje verde "Se asignaron 2 pedidos a Jorge Pérez..."; en la tabla pasan a "Asignado" con "Jorge Pérez" |
| Ir a **Equipo de reparto** | Jorge: "Automóvil OAX-777", "0 de 2" y etiqueta "Con ruta asignada" |
| Ir a **Vehículos** | OAX-777 en "En uso hoy" con "Hoy: Jorge Pérez" |
| Volver a Pedidos y pulsar "Desasignar" en uno | confirmación y vuelve a "Sin asignar" |
| Desasignar el otro | Jorge vuelve a "Libre" en Equipo y OAX-777 a "Disponible" (el recorrido vacío se eliminó) |
| Cambiar la fecha del panel a mañana | las listas se recargan; Luis y Ana ya aparecen libres |
| Fecha de ayer | error bajo la fecha y botón desactivado |
| Como `negocio2@rutaoax.com` | solo ve su pedido (el de Beatriz Núñez) y su propio equipo y vehículos |
Con F12 → Consola no deben salir errores en rojo. Prueba también en ancho móvil: la tabla se desplaza y el panel de asignación queda debajo.

**Limitación conocida de esta parte:** el panel solo ofrece repartidores y vehículos *sin recorrido ese día*, así que desde la pantalla todavía no se pueden sumar pedidos a una ruta que ya existe (el backend sí lo soporta). Queda anotada en `06-ESTADO.md`; se puede resolver en la parte siguiente.

## 6. Cerrar la parte
```powershell
cd ..    # raíz del repo
git add backend bruno
git commit -m "feat(pedidos): crud de pedidos con filtros y asignacion a repartidor y vehiculo"
git add frontend docs
git commit -m "feat(frontend): pantalla de pedidos con panel de asignacion"
git push
```
1. **Tablero:** HU-12 y HU-13 → Done si todo pasó.
2. **`docs/06-ESTADO.md`:** cambia 🟨 por ✅ en HU-12, HU-13 y las filas A13, B8 y A14; borra el problema abierto #1; llena fecha y "Quién". Si ya habías puesto repo, tablero y Figma en tu copia, pásalos al archivo nuevo.
3. **Pendiente:** el despliegue con HTTPS (HU-09) sigue sin ejecutarse. Con el Día 2 casi terminado es buen momento: Parte B de `dia-1/EJECUTAR-PARTE-4.md`, y después `./deploy/deploy.ps1`.

## 7. Qué sigue (Día 2 · Parte 3)
HU-14 (panel del negocio con indicadores) y HU-15 (publicar un pedido y reasignar uno no entregado). Con eso se cierra el Día 2; después empieza el Día 3 (repartidores, cliente, mensajes y comunicaciones reales).
Si sigues en esta conversación no necesito nada; en una nueva, súbele `00-CONTEXTO.md`, `02-API.md`, `03-HISTORIAS.md`, `06-ESTADO.md` y `dia-1/01-INTERFACES-COMPARTIDAS.md` (con todos los anexos).

## 8. Si algo falla
| Síntoma | Qué hacer |
|---|---|
| `mvn -q compile` marca un error | Copia el error completo y el archivo; pide a una IA (con `00-CONTEXTO.md` y los anexos de interfaces) el archivo completo corregido |
| Arranque: `Validation failed for query for method ...` | Pega el mensaje completo y `PedidoRepository` o `RecorridoRepository` |
| Arranque: `Could not resolve parameter` / `named parameter not bound` | Falta o sobra un parámetro en `PedidoRepository.buscar`: pega el mensaje y el archivo |
| El listado de pedidos responde 500 | Primer error en rojo de la consola del backend, con `PedidoService` y `PedidoRepository` |
| Bruno 14 falla (`res.body[0]`) | No hay repartidores libres el 15-06-2030: revisa `Equipo`, que Ana, Jorge y Luis estén activos |
| Bruno 17 devuelve 409 | Quedó un recorrido de pruebas anterior: `SELECT * FROM recorridos WHERE fecha = '2030-06-15';` y bórralo (y vuelve a ejecutar la colección) |
| Bruno 26 devuelve 409 | El recorrido vacío no se eliminó: pasa a una IA `AsignacionService.desasignar` y `RecorridoService.compactar` |
| El panel dice "No hay repartidores libres ese día" | Todos tienen recorrido esa fecha; cambia la fecha del panel |
| Los pedidos no se pueden marcar | Solo se marcan los "Sin asignar" |
| Pantalla en blanco | F12 → Consola: copia el primer error rojo y el archivo que menciona |
Para pedir ayuda a una IA usa siempre: prompt maestro + `00-CONTEXTO.md` + el error completo + solo el archivo que falla.
