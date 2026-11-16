# EJECUTAR LA PARTE 2 · HU-03 (registro) y HU-04 (login) con sus pantallas

**Meta:** entrar a `http://localhost:5173/login` con los usuarios demo, ver cada rol en su sección con su menú, y crear cuentas nuevas desde el registro.
**Tiempo:** 45 a 60 minutos. Comandos en PowerShell.

> El código de esta parte (backend y frontend) se escribió sin poder compilarlo ni ejecutarlo en el entorno donde se generó. Los pasos 3 y 5 son la primera compilación real. Si algo falla, el paso 9 dice qué hacer.

## 1. Antes de copiar: tu cambio al SQL
Tu ajuste a `fk_ped_negocio` es correcto y ya quedó documentado. Este zip trae el `V1__esquema.sql` con esa misma línea; si ya lo tienes igual, no pasa nada al reemplazarlo. Lo más probable de que mi primera corrección "siguiera fallando" es que la base ya tenía tablas creadas por el intento anterior (MySQL no revierte los `CREATE TABLE` cuando falla una migración); en cualquier reintento hay que hacer `DROP DATABASE rutaoax;` y crearla de nuevo.

## 2. Copiar el zip al repositorio
Copia el contenido de `rutaoax-parte2` sobre tu repo (elige *Reemplazar*):
- `backend/...`: **reemplaza** `SecurityConfig.java` y `GlobalExceptionHandler.java` y agrega los archivos de `auth/`, `usuario/` y `config/`.
- `frontend/...`: carpeta nueva completa.
- `docs/`: actualiza `06-ESTADO.md`, `01-BASE-DATOS.md` y `dia-1/01-INTERFACES-COMPARTIDAS.md`.
Comprueba: existe `backend\src\main\java\com\rutaoax\auth\AuthController.java` y `frontend\package.json`.

## 3. Compilar y arrancar el backend
```powershell
cd backend
mvn -q compile
mvn spring-boot:run
```
`compile` no debe mostrar errores. Al arrancar debes ver `Started RutaoaxApplication` y **no** las líneas de los seeders (los datos ya existen). Deja esta terminal abierta.
Verifica que `application.properties` conserve `app.jwt.secret` con **32 caracteres o más** y `app.cors.allowed-origins=http://localhost:5173`.

## 4. Probar el backend con Bruno
Abre la colección `bruno/`, entorno `local`. Ejecuta las peticiones **una por una** en este orden. Hoy solo estas deben salir en verde:
| Petición | Esperado hoy |
|---|---|
| Auth / Login admin, negocio, repartidor empresa, independiente, cliente | 200 y guardan su token |
| Auth / Login incorrecto 401 | 401 |
| Auth / Me con token | 200 con el usuario y su `perfil` |
| Auth / Registro correcto | 201 |
| Auth / Registro contrasena debil 422 | 422 con `errores.password` |
| Auth / Registro correo repetido 409 | 409 |
| Error / Recurso inexistente 404 | 404 en JSON |
| Error / Datos invalidos 422 | 422 con `errores` de nombre, correo, contraseña y tipoCuenta |
| Seguridad / Sin token 401 | 401 en JSON |
| Perfil / Login usuario nuevo | 200 (usa el correo del registro anterior) |
Estas **deben salir en rojo todavía** porque se implementan después: Auth 11 a 15 (logout, token revocado, olvidé contraseña, restablecer), Perfil 1, 2, 3 y 5 (HU-07), y Seguridad 2 a 5 (HU-08: hoy dan 404 en vez de 403).
Comprobación extra en MySQL: `SELECT email, nombre FROM usuarios ORDER BY id DESC LIMIT 1;` muestra el usuario recién registrado en minúsculas.

## 5. Arrancar el frontend
En otra terminal (el backend sigue corriendo):
```powershell
node -v          # v18 o superior
cd frontend
npm install
npm run dev
```
`npm install` tarda un minuto. Debes ver `Local: http://localhost:5173/`. Abre esa dirección.

## 6. Probar las pantallas
| Prueba | Esperado |
|---|---|
| Abrir `http://localhost:5173` | te manda a `/login` (panel azul a la izquierda, formulario a la derecha) |
| Pulsar "Entrar" con todo vacío | mensajes rojos bajo el correo y la contraseña, sin ventanas emergentes |
| Correo `abc` | "El correo no tiene un formato válido" al salir del campo |
| `admin@rutaoax.com` con contraseña incorrecta | mensaje rojo "Correo o contraseña incorrectos" |
| `admin@rutaoax.com` / `Admin#2026` | `/admin/usuarios`: menú de Administrador y tu nombre arriba a la derecha |
| `negocio1@rutaoax.com` / `Ruta#2026` | `/negocio`, menú de 5 opciones, "Negocio · Dulces del Valle" |
| `repartidor1@rutaoax.com` | `/repartidor`, menú Mi ruta y Historial |
| `independiente1@rutaoax.com` | `/independiente`, barra lateral **morada** |
| `cliente1@rutaoax.com` | `/cliente`, menú Mi pedido y Mensajes |
| Recargar la página con sesión iniciada | sigues dentro |
| Como cliente, escribir en la barra `localhost:5173/negocio` | "No tienes permiso para ver esta pantalla" |
| Clic en tu nombre → "Cerrar sesión" | vuelves a `/login` |
| `/registro` con contraseña `12345` | error bajo la contraseña y lista de requisitos sin marcar |
| `/registro` con `admin@rutaoax.com` | mensaje "Ya existe una cuenta con ese correo" |
| Crear una cuenta de cliente nueva | vas a `/login` con un mensaje verde; entras con ella y llegas a `/cliente` |
| Crear una cuenta "Tengo un negocio" (pide nombre del negocio) | al entrar, `/negocio` con "Negocio · el nombre que escribiste" |
| Crear una cuenta "repartidor independiente" | al entrar, `/independiente` morada |
Con el navegador abierto en F12 → Consola no deben aparecer errores en rojo.
Por último: `npm run build` debe terminar sin errores.
Nota: "Cerrar sesión" hoy solo limpia el navegador; el token seguirá válido en el servidor hasta implementar HU-05. Es lo esperado.

## 7. Cerrar la parte
```powershell
cd ..    # raíz del repo
git add backend
git commit -m "feat(auth): login jwt, registro y seguridad base"
git add frontend docs
git commit -m "feat(frontend): layout por rol, login, registro y rutas protegidas"
git push
```
Luego:
1. **Tablero:** HU-03 y HU-04 → Done si todo lo anterior pasó. HU-07 queda en In Progress (parcial).
2. **`docs/06-ESTADO.md`:** cambia 🟨 por ✅ en las filas de HU-03, HU-04, A5, A6, B1, B2 y B3; borra el problema abierto #1; llena fecha y "Quién"; escribe tu repo, tablero y Figma si ya los tienes (el archivo del zip los trae en blanco). Commit: `docs: estado parte 2 verificada`.
3. Lo que falle y no resuelvas, déjalo en 🟨 y descríbelo en "Problemas abiertos".

## 8. Qué sigue (Parte 3)
HU-05 (logout con revocación), HU-06 (recuperar contraseña y servicio de correo, backend y pantallas), HU-08 (seguridad por rol, para que Seguridad 2 a 5 den 403), HU-07 (perfil, backend y pantalla) y HU-09 (primer despliegue con HTTPS).
**Si sigues en esta misma conversación, no necesito que me pases nada**: tengo todo lo generado. Si abres una conversación nueva (o usas otra cuenta de IA), súbele: `00-CONTEXTO.md`, `02-API.md`, `03-HISTORIAS.md`, `06-ESTADO.md` (ya actualizado), `dia-1/01-INTERFACES-COMPARTIDAS.md` (con el anexo) y `dia-1/02-TAREAS-DIA-1.md`, más los archivos que vaya a modificar (para HU-05 y HU-08, `AuthService`, `AuthController`, `JwtFilter` y `SecurityConfig`).

## 9. Si algo falla
| Síntoma | Qué hacer |
|---|---|
| `mvn -q compile` marca `cannot find symbol` u otro error | Copia el error completo y el archivo indicado a una IA con `00-CONTEXTO.md` y el anexo de interfaces; pide el archivo completo corregido |
| Al arrancar: `WeakKeyException` | `app.jwt.secret` mide menos de 32 caracteres |
| Al arrancar: `Could not resolve placeholder 'app.jwt.secret'` o `app.cors.allowed-origins` | Falta esa línea en `application.properties` (cópiala del `.example`) |
| Login responde 401 con la clave correcta | Reinicia el backend; confirma que el seed corrió (`SELECT COUNT(*) FROM usuarios;` = 13 o más) |
| Bruno: `Registro correcto` da 409 | Ejecuta otra vez la petición: genera un correo nuevo en cada intento |
| `npm install` falla o `npm` no se reconoce | Instala Node 20 LTS y abre una terminal nueva |
| El navegador muestra "Network Error" o texto de conexión | El backend no está corriendo o `.env.development` apunta a otro puerto (debe ser 9250) |
| Consola: error de CORS | `app.cors.allowed-origins` debe ser exactamente `http://localhost:5173`; reinicia el backend |
| Pantalla en blanco | Abre F12 → Consola, copia el primer error rojo y el archivo que menciona |
| Tras iniciar sesión vuelve a `/login` | Borra el almacenamiento: F12 → Application → Session Storage → elimina `rutaoax_token` y entra de nuevo |
| `Failed to resolve import` | Falta `npm install` o falta un archivo copiado: revisa que exista la ruta que menciona |
Para pedir ayuda a una IA usa siempre: prompt maestro + `00-CONTEXTO.md` + el error completo + solo el archivo que falla.
