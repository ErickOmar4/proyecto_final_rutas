# EJECUTAR LA PARTE 1 · HU-01 y HU-02 (desde cero)

**Meta:** el backend arranca en `http://localhost:9250`, Flyway crea las 12 tablas, el seed carga los datos de prueba y cualquier error responde con el JSON del contrato.
**Tiempo:** 40 a 60 minutos la primera vez. Comandos en **PowerShell** (Windows) con VS Code.

> Importante: el código de esta parte se escribió sin poder compilarlo (el entorno donde se generó no tiene Maven ni internet). El paso 5 es la primera compilación real. Si algo falla, el paso 11 dice qué hacer.

## 1. Requisitos (comprueba cada uno)
| Comando | Debes ver |
|---|---|
| `java -version` | `openjdk version "21...` (o Temurin 21) |
| `mvn -v` | `Apache Maven 3.9...` y `Java version: 21` |
| `mysql --version` | contiene **MySQL** y `8.` (si dice MariaDB, no sirve) |
| `git --version` | cualquier versión |
Si falta Java o Maven: instala JDK 21 y Maven 3.9, y agrega sus carpetas `bin` al `PATH`; abre una terminal nueva. También instala Bruno.

## 2. Crear el repositorio y copiar los paquetes
```powershell
mkdir rutaoax; cd rutaoax
git init -b main
```
Copia dentro de `rutaoax/`, **en este orden** (si Windows pregunta, elige *Reemplazar*):
1. el contenido del zip `rutaoax-plan` (carpetas `docs/`, `deploy/`, `backend/`, `diseno/`),
2. el contenido del zip `rutaoax-dia1` (agrega `docs/dia-1/`, `bruno/` y actualiza `application.properties.example`),
3. el contenido del zip **`rutaoax-parte1`** (agrega el código de `backend/` y actualiza `docs/00-CONTEXTO.md`, `03`, `06-ESTADO.md` y `08`; agrega esta guía).

Comprueba: `dir backend\src\main\java\com\rutaoax` debe listar `auth, common, config, mensaje, negocio, notificacion, pedido, recorrido, usuario, vehiculo` y `RutaoaxApplication.java`.

## 3. Base de datos local
```powershell
mysql -u root -p
```
```sql
CREATE DATABASE rutaoax CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
CREATE USER 'rutaoax_app'@'localhost' IDENTIFIED BY 'Rutaoax_local1';
GRANT ALL PRIVILEGES ON rutaoax.* TO 'rutaoax_app'@'localhost';
FLUSH PRIVILEGES;
EXIT;
```
Prueba: `mysql -u rutaoax_app -pRutaoax_local1 rutaoax -e "SELECT 1;"` → tabla con un `1`.

## 4. Configuración local
```powershell
cd backend\src\main\resources
copy application.properties.example application.properties
notepad application.properties
```
Cambia solo estas líneas (el resto déjalo igual):
```
spring.datasource.username=rutaoax_app
spring.datasource.password=Rutaoax_local1
app.jwt.secret=rutaoax-clave-local-de-desarrollo-minimo-32-caracteres
```
`application.properties` está en `.gitignore`: no se sube a Git.

## 5. Compilar
```powershell
cd ..\..\..    # queda en backend\
mvn -q compile
```
Debes ver **nada** (sin errores) y volver al prompt. La primera vez tarda porque descarga dependencias.

## 6. Arrancar
```powershell
mvn spring-boot:run
```
Busca estas líneas en la consola (el orden puede variar un poco):
- `Successfully applied 1 migration to schema `rutaoax``
- `SeederUsuarios: 13 usuarios, 2 negocios y 6 repartidores creados`
- `SeederOperacion: 10 vehículos, 6 recorridos, 15 pedidos y 10 mensajes creados`
- `Tomcat started on port 9250`
- `Started RutaoaxApplication`
Deja esta terminal abierta. (Verás una línea "Using generated security password": es normal, no se usa.)

## 7. Verificar la base (HU-01)
En otra terminal: `mysql -u rutaoax_app -pRutaoax_local1 rutaoax`
```sql
SHOW TABLES;                                   -- 13 filas (12 tablas + flyway_schema_history)
SELECT COUNT(*) FROM usuarios;                 -- 13
SELECT COUNT(*) FROM repartidores;             -- 6
SELECT COUNT(*) FROM vehiculos;                -- 10
SELECT COUNT(*) FROM recorridos;               -- 6
SELECT COUNT(*) FROM pedidos;                  -- 15
SELECT COUNT(*) FROM mensajes;                 -- 10
SELECT estado, COUNT(*) FROM pedidos GROUP BY estado;
-- PENDIENTE 7, ASIGNADO 3, EN_CAMINO 2, ENTREGADO 2, NO_ENTREGADO 1
SELECT email, LEFT(password_hash, 4) FROM usuarios LIMIT 3;   -- empieza con $2a$
SELECT u.email, r.nombre FROM usuarios u JOIN usuario_roles ur ON ur.usuario_id=u.id JOIN roles r ON r.id=ur.rol_id;   -- 13 filas, un rol cada uno
SELECT modalidad, COUNT(*) FROM pedidos GROUP BY modalidad;   -- ABIERTO 5, NEGOCIO 10
SELECT lat, lng FROM pedidos LIMIT 3;          -- coordenadas cerca de 17.0 y -96.7
```
Haz una copia rápida del estado: no hace falta, el seed se puede repetir borrando la base.

## 8. Verificar los errores JSON (HU-02)
Con el backend corriendo, en otra terminal:
```powershell
curl.exe -i http://localhost:9250/api/ruta-que-no-existe
```
Debes ver `HTTP/1.1 404` y un JSON como:
```json
{"timestamp":"2026-11-15T10:00:00","status":404,"error":"Not Found","message":"El recurso solicitado no existe","path":"/api/ruta-que-no-existe"}
```
En Bruno (abre la carpeta `bruno/`, entorno `local`): `Error/Recurso inexistente 404` → en verde.
Los casos de 422, 401 y 409 se prueban cuando existan los endpoints (Parte 2).

## 9. Probar que el seed no se duplica
Detén el backend (`Ctrl + C`), vuelve a arrancar (`mvn spring-boot:run`) y repite `SELECT COUNT(*) FROM usuarios;` → sigue en **13**. En la consola no aparecen las líneas de los seeders.

## 10. Revisar el mapeo de las entidades (una sola vez)
Como `ddl-auto=none` no avisa de errores de mapeo hasta que se usa una tabla, haz esta prueba:
1. En `application.properties` cambia temporalmente `spring.jpa.hibernate.ddl-auto=none` por `validate`.
2. Arranca el backend. Si arranca → mapeo correcto.
3. Si falla con `missing column` o `missing table`: es un error real; copia el mensaje completo y ve al paso 11.
4. Si falla solo por el tipo de una columna `boolean`/`tinyint`/`bit` o `decimal`: ignóralo.
5. **Regresa a `none`.**

## 11. Si algo falla
| Síntoma | Qué hacer |
|---|---|
| `'mvn' no se reconoce` | Maven no está en el `PATH`; ábrelo de nuevo tras agregarlo |
| `release version 21 not supported` | `JAVA_HOME` apunta a otro JDK; debe ser el 21 |
| `cannot find symbol` / error de compilación | Copia el error completo y el archivo indicado a una IA con `00-CONTEXTO.md` y `01-INTERFACES-COMPARTIDAS.md`; pide el archivo completo corregido |
| `Access denied for user` | usuario o clave de `application.properties` no coinciden con el paso 3 |
| `Unknown database 'rutaoax'` | falta el `CREATE DATABASE` del paso 3 |
| `Communications link failure` | el servicio MySQL no está encendido (`net start MySQL80`) |
| `Port 9250 was already in use` | otra instancia sigue corriendo; ciérrala |
| `Flyway ... failed migration` o `checksum mismatch` | solo en local: `DROP DATABASE rutaoax;` y repite el paso 3 y el 6 |
| `Table 'rutaoax.xxx' doesn't exist` | Flyway no corrió: revisa `spring.flyway.enabled=true` |
| `Schema-validation: missing column` (paso 10) | error de mapeo: pasa el mensaje, la entidad y `V1__esquema.sql` a una IA |
| Falla el seed con `Duplicate entry` | quedó la base a medias: `DROP DATABASE rutaoax;` y repite |

Para pedir ayuda a una IA usa siempre: prompt maestro + `00-CONTEXTO.md` + el error completo + solo el archivo que falla.

## 12. Cerrar la parte
```powershell
cd ..    # raíz del repo
git add .
git commit -m "feat(backend): esqueleto, flyway, entidades, seeders y errores globales"
git remote add origin https://github.com/TU_USUARIO/rutaoax.git
git push -u origin main
```
(Si ya tienes remoto, omite `git remote add`.)
Luego:
1. **Tablero:** HU-01 y HU-02 → Done.
2. **`docs/06-ESTADO.md`:** cambia 🟨 por ✅ en HU-01, HU-02 y en las filas A1 a A4; borra el problema abierto #1; llena fecha y "Quién"; si cambiaste algo, anótalo en "Decisiones". Commit: `docs: estado parte 1 verificada`.
3. Si alguna comprobación falló y no la resolviste, déjala en 🟨 y descríbela en "Problemas abiertos".

## 13. Qué sigue (Parte 2)
HU-04 (login con JWT, reemplaza `SecurityConfig`), HU-03 (registro) y las pantallas de login y registro del frontend, más la colección de Bruno con token. Cuando la Parte 1 esté verificada, pásame el resultado de los pasos 6, 7 y 8 y preparo la Parte 2 con los `06-ESTADO.md` actualizados.
