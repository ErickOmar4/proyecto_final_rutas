# EJECUTAR LA PARTE 3 · HU-05 (logout), HU-06 (recuperar contraseña) y HU-08 (seguridad por rol)

**Meta:** cerrar sesión invalida el token en el servidor, se puede recuperar la contraseña con un enlace, y cada rol solo entra a sus rutas.
**Tiempo:** 45 a 60 minutos. Comandos en PowerShell.

> Igual que en las partes anteriores, este código se escribió sin poder compilarlo ni ejecutarlo en el entorno donde se generó. Los pasos 3 y 5 son la primera prueba real; el paso 9 dice qué hacer si algo falla.

## 1. Copiar el zip al repositorio
Copia el contenido de `rutaoax-parte3` sobre tu repo (elige *Reemplazar*):
- `backend/`: **reemplaza** `AuthController.java`, `AuthService.java`, `SecurityConfig.java` y `PasswordResetTokenRepository.java`; agrega `PasswordResetService`, `TokenCleanupJob`, `EmailService`, `AsyncConfig`, `SchedulingConfig` y los dos DTO nuevos. También trae un `application.properties.example` actualizado.
- `frontend/`: reemplaza `RecuperarPage.jsx` y `RestablecerPage.jsx`.
- `bruno/`: agrega 2 peticiones a `Auth` y 4 a `Seguridad`.
- `docs/`: actualiza `06-ESTADO.md` y `dia-1/01-INTERFACES-COMPARTIDAS.md`.

## 2. Ajustar tu `application.properties` local
Tu `application.properties` no se versiona, así que no se actualiza solo. Agrega estas líneas debajo de `spring.mail.port=25` (evitan que un correo lento deje colgado el servidor):
```
spring.mail.properties.mail.smtp.connectiontimeout=5000
spring.mail.properties.mail.smtp.timeout=5000
spring.mail.properties.mail.smtp.writetimeout=5000
```
Para probar la recuperación **sin correo real**, cambia o agrega esta línea (solo en local):
```
app.dev.log-reset-link=true
```
Comprueba que existan `app.mail.from=no-reply@devs8.tech` y `app.frontend-url=http://localhost:5173`.

## 3. Compilar y arrancar el backend
```powershell
cd backend
mvn -q compile
mvn spring-boot:run
```
`compile` no debe mostrar errores. Debes ver `Started RutaoaxApplication`. Si falla el arranque, copia el bloque `APPLICATION FAILED TO START` (paso 9).

## 4. Probar el backend con Bruno
Entorno `local`. Ejecuta toda la carpeta **Auth** y luego **Seguridad**. Esperado hoy:
| Petición | Esperado |
|---|---|
| Auth 1 a 10 (logins, me, registros) | igual que antes |
| Auth 11 Login temporal para logout | 200 |
| Auth 12 Logout | **204** |
| Auth 13 Token revocado 401 | **401** (el mismo token ya no sirve) |
| Auth 14 Olvide mi contrasena | **200** |
| Auth 15 Restablecer token invalido 409 | **409** |
| Auth 16 Olvide contrasena correo inexistente | **200** (no revela si el correo existe) |
| Auth 17 Restablecer contrasena debil 422 | **422** con `errores.password` |
| Seguridad 1 a 9 | todas en verde: la primera da 401 y las demás **403** |
| Perfil 4 Login usuario nuevo | 200 |
Siguen en rojo hasta la Parte 4 (HU-07): Perfil 1, 2, 3 y 5.

## 5. Verificar en MySQL
```sql
SELECT jti, expira_en FROM tokens_revocados;                       -- una fila por cada logout que hiciste
SELECT usuario_id, LEFT(token_hash, 12) AS hash, LENGTH(token_hash) AS largo, expira_en, usado
  FROM password_reset_tokens;                                       -- largo 64; usado 0 en el más reciente
SELECT canal, evento, estado, destino, LEFT(detalle, 70) AS detalle
  FROM notificaciones_log ORDER BY id DESC LIMIT 10;
```
En `notificaciones_log` verás filas con canal `EMAIL` y eventos `REGISTRO` y `RECUPERACION`. **Hoy el estado esperado es `FALLIDO`** (no hay un servidor de correo en tu PC) con un detalle como "Mail server connection failed". Eso es correcto: el registro y la recuperación siguen funcionando y nunca se finge un envío. Pasará a `ENVIADO` cuando exista Postfix en el VPS.

## 6. Probar las pantallas
Arranca el frontend si no lo tienes (`cd frontend; npm run dev`). **Para el flujo de recuperación usa una cuenta de prueba que crees en `/registro` (por ejemplo `prueba1@rutaoax.com`), no los usuarios demo**, para no cambiar sus contraseñas.
| Prueba | Esperado |
|---|---|
| Entrar con cualquier usuario → menú de usuario → "Cerrar sesión" | vuelves a `/login` y en MySQL aparece un nuevo `jti` en `tokens_revocados` |
| Pulsar atrás del navegador tras cerrar sesión | te manda de nuevo a `/login` |
| `/login` → "¿Olvidaste tu contraseña?" | abre `/recuperar` |
| `/recuperar` con el correo vacío o `abc` | error bajo el campo |
| `/recuperar` con `prueba1@rutaoax.com` | mensaje verde: "Si el correo está registrado, te enviamos un enlace…" |
| Mirar la consola del backend | línea `[DESARROLLO] Enlace de recuperación para prueba1@rutaoax.com: http://localhost:5173/restablecer?token=...` |
| Abrir ese enlace | pantalla "Crea tu contraseña nueva" con la lista de requisitos |
| Contraseña `12345` o confirmación distinta | errores bajo cada campo |
| Contraseña válida (ej. `Nueva#2026`) en ambos campos | mensaje verde "Tu contraseña se cambió correctamente" |
| Entrar con la contraseña vieja | "Correo o contraseña incorrectos" |
| Entrar con la nueva | entra a `/cliente` |
| Abrir otra vez el **mismo** enlace y enviar | error "El enlace no es válido o ya venció" y botón "Pedir un enlace nuevo" |
| `/restablecer` sin `?token=` | mensaje de enlace inválido |
| Pedir dos enlaces seguidos y usar el primero | el primero ya no sirve (solo vale el más reciente) |
Con F12 → Consola no deben salir errores en rojo.
Seguridad en la interfaz: como `cliente1@rutaoax.com`, abre `localhost:5173/negocio` → "No tienes permiso" (esto ya lo hacía el frontend; ahora además el servidor responde 403 si alguien llama a la API directamente: lo viste en Bruno).

## 7. Cerrar la parte
```powershell
cd ..
git add backend bruno
git commit -m "feat(auth): logout con revocacion, recuperacion de contrasena y seguridad por rol"
git add frontend docs
git commit -m "feat(frontend): recuperar y restablecer contrasena"
git push
```
Luego:
1. **Tablero:** HU-05, HU-06 y HU-08 → Done si todo pasó. Si el correo real es lo único pendiente, HU-06 puede quedar en Review con un comentario ("falta Postfix, se cierra en HU-26").
2. **`docs/06-ESTADO.md`:** cambia 🟨 por ✅ en HU-05, HU-06, HU-08 y en las filas A7, A8, A9 y B4; borra el problema abierto #1 (deja el #3 del correo); llena fecha y "Quién". Si ya habías escrito repo, tablero y Figma en tu copia anterior, pásalos al archivo nuevo. Commit: `docs: estado parte 3 verificada`.

## 8. Qué sigue (Parte 4)
HU-07 (perfil: endpoints, pantalla `/perfil` y cambio de contraseña con confirmación) y HU-09 (primer despliegue en el VPS con HTTPS). Para el despliegue conviene que tengas hechos los pasos 1 a 6 de `05-DESPLIEGUE.md` (DNS, Java, MySQL, Nginx, Certbot); si te falta alguno, dímelo y lo preparamos juntos.
Si sigues en esta conversación no necesito que me pases nada; en una conversación nueva súbele `00-CONTEXTO.md`, `02-API.md`, `03-HISTORIAS.md`, `06-ESTADO.md` y `dia-1/01-INTERFACES-COMPARTIDAS.md` (con los anexos), y `05-DESPLIEGUE.md` para la HU-09.

## 9. Si algo falla
| Síntoma | Qué hacer |
|---|---|
| Error de compilación | Copia el error completo y el archivo que menciona; pídelo corregido completo a una IA con `00-CONTEXTO.md` y los anexos de interfaces |
| Arranque: `No qualifying bean of type 'JavaMailSender'` | Falta `spring.mail.host` en `application.properties` (cópialo del `.example`) |
| Arranque: `Could not resolve placeholder 'app.mail.from'` o `app.frontend-url` | Falta esa línea en `application.properties` |
| Logout responde 401 | El token ya estaba revocado o vencido: inicia sesión de nuevo |
| Bruno `Token revocado 401` da 200 | El logout no guardó el `jti`: revisa `SELECT * FROM tokens_revocados;` y la consola del backend |
| `forgot-password` responde 500 | Revisa la consola del backend; suele ser `password_reset_tokens` o `usado` mal mapeado: pega el error completo |
| No aparece el enlace en la consola | `app.dev.log-reset-link=true` no está en tu `application.properties` o no reiniciaste el backend; el correo debe existir y estar activo |
| El correo queda `ENVIADO` pero no llega | En local no debería pasar; si pasa, hay un servidor SMTP en el puerto 25 de tu PC |
| Bruno `Seguridad` da 404 en lugar de 403 | Ese usuario sí tiene permiso a la ruta (no existe todavía el controlador); revisa el token que usa la petición |
| Una petición permitida ahora da 403 | La regla de `SecurityConfig` no incluye ese rol; compara con la matriz de `02-API.md` |
| Frontend: "Failed to resolve import" | Falta un archivo copiado; revisa que existan las dos páginas de `pages/auth` |
Para pedir ayuda a una IA usa siempre: prompt maestro + `00-CONTEXTO.md` + el error completo + solo el archivo que falla.
