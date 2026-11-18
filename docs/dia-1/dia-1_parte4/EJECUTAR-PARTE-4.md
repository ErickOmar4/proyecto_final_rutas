# EJECUTAR LA PARTE 4 · HU-07 (perfil) y HU-09 (primer despliegue con HTTPS)

**Meta:** el perfil funciona en tu PC, y `https://rutaoax.devs8.tech` muestra el sistema en el VPS con candado válido y los 5 usuarios demo entrando a su sección.
**Tiempo:** 30 min el perfil + 45 a 75 min el despliegue (casi todo es esperar al servidor). Comandos en PowerShell.

> Como en las partes anteriores, este código y los scripts se escribieron sin poder ejecutarlos. Los pasos 2, 3 y 8 son la primera prueba real; el paso 11 dice qué hacer si algo falla.

---
# PARTE A · HU-07 · Perfil (en tu PC)

## 1. Copiar el zip
Copia el contenido de `rutaoax-parte4` sobre tu repo (elige *Reemplazar*):
- `backend/`: agrega `PerfilController`, `PerfilService` y dos DTO en `usuario/`.
- `frontend/`: agrega `api/perfilApi.js` y `pages/perfil/PerfilPage.jsx`, y **reemplaza `App.jsx`** (solo cambia la ruta `/perfil`).
- `deploy/`: agrega `preparar-servidor.sh` y reemplaza `deploy.ps1`.
- `docs/`: actualiza `06-ESTADO.md`, `05-DESPLIEGUE.md` y `dia-1/01-INTERFACES-COMPARTIDAS.md`.

## 2. Compilar y arrancar
```powershell
cd backend
mvn -q compile
mvn spring-boot:run
```
Sin errores de compilación y `Started RutaoaxApplication`.

## 3. Probar con Bruno (entorno `local`)
Ejecuta toda la colección. **Ahora todo debe salir en verde** salvo nada: las carpetas `Auth`, `Error`, `Perfil` y `Seguridad` completas.
| Petición | Esperado |
|---|---|
| Perfil 1 Ver perfil | 200 |
| Perfil 2 Actualizar perfil | 200 |
| Perfil 3 Cambiar contraseña actual incorrecta | 409 |
| Perfil 4 Login usuario nuevo | 200 |
| Perfil 5 Cambiar contraseña correcta | 204 |
Si alguna de Auth 11 a 17 o Seguridad falla ahora y antes pasaba, avísame con el error.
Nota: Perfil 2 cambia el nombre y teléfono de `cliente1` (queda Carla Ríos, 9515550110); no afecta el inicio de sesión.

## 4. Probar la pantalla `/perfil`
Con `npm run dev` corriendo, entra con una **cuenta de prueba creada en `/registro`** (para no tocar contraseñas demo).
| Prueba | Esperado |
|---|---|
| Clic en tu nombre → "Mi perfil" | pantalla con dos tarjetas: "Mis datos" y "Cambiar contraseña" |
| El correo | aparece bloqueado (no se puede editar) |
| Borrar el nombre y salir del campo | error rojo bajo el nombre |
| Teléfono `123` | "El teléfono debe tener 10 dígitos" |
| Avatar `hola` | error de URL; con `https://i.pravatar.cc/100` la vista previa muestra una imagen |
| Guardar con datos válidos | mensaje verde y el nombre/avatar de la barra superior se actualizan sin recargar |
| Cambiar contraseña con la actual incorrecta | se abre el cuadro de confirmación; al confirmar, mensaje rojo "La contraseña actual es incorrecta" y error bajo ese campo |
| Contraseña nueva `12345` | errores y requisitos sin marcar; no abre la confirmación |
| Actual correcta + nueva válida + confirmación igual | cuadro "¿Seguro que quieres cambiar tu contraseña?" → "Sí, cambiarla" → mensaje verde y campos vacíos |
| Cerrar sesión y entrar con la nueva | funciona |
| Detener el backend y pulsar Guardar | mensaje de error de conexión, sin ventanas emergentes |
Cierra la parte A con: `git add . ; git commit -m "feat(perfil): ver, editar y cambiar contrasena"`.

---
# PARTE B · HU-09 · Primer despliegue (VPS 179.197.84.34)

## 5. Revisa los requisitos (2 minutos)
| Comprobación | Cómo | Esperado |
|---|---|---|
| Registro DNS | en el panel donde administras `devs8.tech`, crea un registro **A**: nombre `rutaoax`, valor `179.197.84.34` | guardado. Si usas Cloudflare, déjalo en "solo DNS" (nube gris) |
| El DNS ya resuelve | `nslookup rutaoax.devs8.tech` | `Address: 179.197.84.34` (puede tardar unos minutos) |
| Acceso por SSH | `ssh root@179.197.84.34 "echo ok"` | te pide la contraseña y responde `ok` |
Si `nslookup` aún no resuelve, puedes continuar con el paso 6; el certificado se pide al final cuando ya resuelva.

**Recomendado (evita escribir la contraseña 4 veces en cada publicación):** crea una llave SSH una vez.
```powershell
ssh-keygen -t ed25519
type $env:USERPROFILE\.ssh\id_ed25519.pub | ssh root@179.197.84.34 "mkdir -p ~/.ssh && cat >> ~/.ssh/authorized_keys"
ssh root@179.197.84.34 "echo ok"     # ahora responde sin pedir contraseña
```
(En `ssh-keygen` pulsa Enter en todo.)

## 6. Preparar el servidor (una sola vez)
Desde la raíz del repo:
```powershell
scp deploy/preparar-servidor.sh root@179.197.84.34:/root/
ssh root@179.197.84.34 'sed -i "s/\r$//" /root/preparar-servidor.sh; CERTBOT_EMAIL=tucorreo@ejemplo.com bash /root/preparar-servidor.sh'
```
(Cambia `tucorreo@ejemplo.com` por tu correo real; Let's Encrypt lo usa para avisos de vencimiento.)
Tarda 3 a 8 minutos. Verás 7 secciones numeradas. Debes ver, en orden:
1. `openjdk version "21...` y `mysql  Ver 8.0...` (si aparece **MariaDB**, el script se detiene).
2. `Status: active` del firewall.
3. `base de datos lista`.
5. `Se creó /opt/rutaoax/config/application.properties con claves generadas automáticamente.`
6. `nginx: configuration file ... test is successful`.
7. `rutaoax.devs8.tech apunta a: 179.197.84.34` y, si pasaste `CERTBOT_EMAIL`, `Certificado HTTPS instalado.`
Termina con `LISTO. Servidor preparado.`
Si el DNS aún no resolvía, el script lo avisa: espera unos minutos y repite **solo el segundo comando**; es seguro ejecutarlo de nuevo (no borra nada).

## 7. Probar que Nginx y HTTPS responden
```powershell
curl.exe -I https://rutaoax.devs8.tech
```
Todavía no hay frontend publicado, así que es normal ver `403` o `404`; lo importante es que la conexión **TLS funcione** (sin error de certificado). Si dice `curl: (60) SSL certificate problem`, el certificado no se instaló: repite el paso 6 con `CERTBOT_EMAIL`.

## 8. Publicar la aplicación
```powershell
./deploy/deploy.ps1
```
Hace 4 pasos con su mensaje: compila el backend, compila el frontend, sube los archivos y reinicia el servicio. Debe terminar con `active` y `Listo: https://rutaoax.devs8.tech`.
La primera vez el backend tarda ~20 segundos en crear las tablas y cargar los datos de prueba en la base del servidor.

## 9. Verificar en producción
```powershell
curl.exe -i https://rutaoax.devs8.tech/api/auth/me
ssh root@179.197.84.34 "systemctl is-active rutaoax; mysql -u rutaoax_app -p`$(cat /root/.rutaoax_db_pass) rutaoax -e 'SELECT COUNT(*) AS usuarios FROM usuarios;'"
```
- Primer comando: `HTTP/1.1 401` con un JSON (`"message":"Debes iniciar sesión para continuar"`).
- Segundo: `active` y `usuarios 13`.
En el navegador (ventana privada):
| Prueba | Esperado |
|---|---|
| `https://rutaoax.devs8.tech` | login con **candado** |
| `http://rutaoax.devs8.tech` | te redirige a `https://` |
| `admin@rutaoax.com` / `Admin#2026` | `/admin/usuarios` |
| `negocio1@`, `repartidor1@`, `independiente1@`, `cliente1@rutaoax.com` / `Ruta#2026` | cada uno en su sección (`/negocio`, `/repartidor`, `/independiente`, `/cliente`) |
| Recargar `https://rutaoax.devs8.tech/negocio` con sesión | no sale 404: la app carga |
| Cerrar sesión y usar el mismo token en Bruno (`Token revocado 401`) | 401 |
En Bruno cambia el entorno a **produccion** y ejecuta toda la colección: debe quedar en verde igual que en local.
Prueba de reinicio (opcional pero recomendada): `ssh root@179.197.84.34 "reboot"`, espera 2 minutos y recarga el sitio: debe volver solo.

## 10. Cierre del Día 1
Con todo lo anterior, revisa `docs/dia-1/03-CIERRE-DIA-1.md` (lista de criterios). Después:
```powershell
git add .
git commit -m "feat(perfil): perfil y primer despliegue con https"
git push
```
1. **Tablero:** HU-07 y HU-09 → Done. Con eso HU-01 a HU-09 quedan en Done (HU-06 puede llevar un comentario: "falta Postfix, se cierra en HU-26").
2. **`docs/06-ESTADO.md`:** cambia 🟨 por ✅ en HU-07, HU-09 y las filas A10, B5 y D; borra el problema abierto #1; llena fecha y "Quién". Commit: `docs: cierre dia 1`.
3. **Respaldo de la base de producción:** `ssh root@179.197.84.34 "mysqldump -u rutaoax_app -p`$(cat /root/.rutaoax_db_pass) rutaoax > /root/rutaoax_dia1.sql"`.
4. Anota en `06-ESTADO.md` cualquier ajuste que hayas hecho en el servidor.
Siguiente: **Día 2 (Negocio)**: vehículos, equipo, pedidos con filtros y asignación. Pídemelo en este chat con el mismo formato.

## 11. Si algo falla
| Síntoma | Qué hacer |
|---|---|
| `ssh` pide la contraseña y la rechaza | Usa la contraseña de `root` del panel del proveedor del VPS |
| `ssh: connect to host ... Connection timed out` | El firewall del proveedor bloquea el puerto 22 o la IP es otra; revisa el panel del VPS |
| El script falla en `apt-get install ... openjdk-21` | Pega el error a una IA junto con `cat /etc/os-release`; puede requerir otro paquete en esa versión del sistema |
| El script dice que se instaló MariaDB | Para: este proyecto exige MySQL 8. Avísame con la salida de `cat /etc/os-release` |
| `certbot` falla con "Timeout during connect" o "unauthorized" | El DNS aún no apunta al servidor o hay un firewall del proveedor sin abrir 80/443. Espera, revisa `nslookup`, y repite el paso 6 |
| `scp` o `ssh` dentro de `deploy.ps1` fallan | Prueba cada comando a mano; revisa que el paso 5 pase |
| `deploy.ps1`: "la ejecución de scripts está deshabilitada" | `Set-ExecutionPolicy -Scope CurrentUser RemoteSigned` y vuelve a ejecutarlo |
| El servicio no queda `active` | `ssh root@179.197.84.34 "journalctl -u rutaoax -n 80 --no-pager"` y pega el primer error a una IA con `00-CONTEXTO.md` |
| `502 Bad Gateway` en el navegador | El backend no está corriendo (mismo comando de arriba) |
| Pantalla en blanco al recargar una ruta | Revisa que Nginx tenga `try_files $uri /index.html;` (el script lo trae) |
| Error de CORS en la consola del navegador | `app.cors.allowed-origins` en el servidor debe ser `https://rutaoax.devs8.tech` (el script lo configura) |
| `Access denied for user 'rutaoax_app'` en los logs | La clave de `/opt/rutaoax/config/application.properties` no coincide con `/root/.rutaoax_db_pass`; borra el `application.properties` del servidor y repite el paso 6 (genera uno nuevo) |
| Los usuarios demo no existen en producción | El seed solo corre con la base vacía; mira `journalctl` para ver si falló la creación de tablas |
Para pedir ayuda a una IA usa siempre: prompt maestro + `00-CONTEXTO.md` + el error completo + solo el archivo que falla.
