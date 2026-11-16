# 07 · PROMPTS PARA TRABAJAR CON IA

La IA no recuerda nada entre conversaciones. Por eso **cada chat empieza con el prompt maestro + los documentos que corresponden**. No pases todo el proyecto: pasa solo lo de la tabla.

## 1. Qué adjuntar según la tarea
| Tipo de tarea | Adjuntar siempre | Además |
|---|---|---|
| Backend de un módulo | `00-CONTEXTO.md`, la sección del módulo en `02-API.md`, la tarjeta de `03-HISTORIAS.md` | `V1__esquema.sql` (solo las tablas del módulo), las entidades y repositorios que ya existen del módulo, y `SecurityConfig` si toca permisos |
| Pantalla de React | `00-CONTEXTO.md`, la sección de la API que consume, la tarjeta de la HU | el HTML de esa pantalla de `diseno/`, `estilos.css`, `http.js`, `AuthContext.jsx`, y los componentes de `components/ui` que use |
| Corregir un error | `00-CONTEXTO.md` | el mensaje de error completo (consola o `journalctl`), y solo el/los archivos donde falla |
| Despliegue o servidor | `05-DESPLIEGUE.md` | salida exacta del comando que falla y `cat /etc/os-release` |
| Bruno | `02-API.md` (secciones a probar) | un archivo `.bru` existente como ejemplo de formato |
| README o documentación | `00`, `01`, `02` | el README actual |

## 2. PROMPT MAESTRO (pégalo al inicio de cada chat)
```
Eres un desarrollador senior de Spring Boot (Java 21), React y MySQL. Trabajas en RutaOax, un proyecto que ya está en marcha: NO empiezas desde cero y NO tienes el repositorio completo. Solo conoces lo que te adjunto en este chat.

Antes de escribir código:
1. Lee el contexto, el contrato de API y la tarjeta de la tarea que te adjunto.
2. Si te falta un archivo o dato para hacer la tarea bien, pídemelo en una lista corta y espera. No lo adivines.

Reglas obligatorias:
- Respeta al pie de la letra los nombres de rutas, campos JSON, tablas y paquetes de los documentos. No inventes endpoints, tablas ni campos. Si crees que hace falta uno nuevo, propónlo aparte y espera mi autorización.
- No cambies la arquitectura ni toques archivos que no pertenezcan a esta tarea.
- No dupliques clases o componentes que ya existen en lo que te adjunté; reutilízalos.
- Nunca devuelvas entidades JPA: usa DTOs. Validaciones solo con Bean Validation. Errores con el formato JSON del contrato.
- En React: sin alert() ni confirm(); validaciones bajo cada input; estados de carga, vacío y error de red; modal para acciones destructivas.
- No uses datos simulados donde debe haber conexión real a la API o a la base de datos.
- No simules un envío de correo, SMS o WhatsApp.

Formato de tu respuesta:
1. Lista de archivos NUEVOS y MODIFICADOS (con ruta completa).
2. El código COMPLETO de cada archivo (no fragmentos, no "...resto igual"), cada uno en su bloque con la ruta como título.
3. Comandos exactos para compilar y probar, y qué debería ver si todo va bien.
4. La petición de Bruno (método, URL, cuerpo y respuesta esperada) que demuestra la tarea.
5. Una lista de verificación con cada criterio de aceptación de la tarjeta, indicando cómo se cumple.
6. Dependencias o problemas pendientes con otros módulos.
No declares la tarea terminada si algún criterio de aceptación queda sin cubrir; dilo explícitamente.
```

## 3. PLANTILLA DE TAREA (pégala después del prompt maestro y de los documentos)
```
TAREA: HU-__ · <nombre>
Contexto adjunto: 00-CONTEXTO.md, 02-API.md (sección ____), 03-HISTORIAS.md (tarjeta HU-__)
Archivos existentes que debes modificar o usar (adjuntos abajo): ____
Archivos que debes crear: ____
No toques: ____
Hazlo en este orden: ____
```

## 4. Prompts rápidos listos para copiar
**Backend de un módulo**
```
Implementa el backend de la HU-__ (módulo ____): entidad (si falta), repositorio con consultas paginadas y JOIN FETCH, servicio con las reglas y comprobación de pertenencia, controlador con @PreAuthorize, DTOs de entrada con Bean Validation y DTOs de salida con método estático from(). Usa exactamente las rutas de 02-API.md. Entrégame los archivos completos y las peticiones de Bruno.
```
**Pantalla de React**
```
Implementa la pantalla de la HU-__ en React reproduciendo el diseño HTML adjunto con las clases de estilos.css (no inventes otro estilo). Crea el archivo api/____Api.js con las llamadas del contrato, la página, y los componentes que falten. Tabla con paginación del servidor y filtros enviados como parámetros. Estados de carga, vacío y error. Entrégame los archivos completos y cómo agregar la ruta en el router.
```
**Corregir un error**
```
Tengo este error al probar la HU-__: <pegar error completo>. Estos son los archivos involucrados: <adjuntos>. Explica la causa en una frase y entrégame solo los archivos que cambian, completos. No cambies nada más.
```
**Revisión antes de hacer commit**
```
Revisa el código que te adjunto contra los criterios de aceptación de la HU-__ y contra las reglas de 00-CONTEXTO.md. Lista únicamente lo que incumple o puede fallar (permisos, validaciones, N+1, datos ajenos expuestos, falta de estados de carga/error). No reescribas lo que está bien.
```
**Revisión de permisos (día 4)**
```
Te adjunto SecurityConfig y los controladores/servicios de ____. Para cada endpoint dime qué roles pueden llamarlo hoy y compáralo con la matriz de 02-API.md. Señala cualquier endpoint donde un usuario pueda ver o modificar datos que no son suyos.
```
**Cerrar sesión de trabajo (al terminar cada chat)**
```
Con lo que hicimos en esta conversación, redáctame las líneas para actualizar 06-ESTADO.md: HU y su estado, archivos creados o modificados, decisiones tomadas y problemas abiertos. Máximo 15 líneas.
```
**Generar la colección de Bruno de una HU**
```
Genera los archivos .bru (formato de Bruno) de las peticiones de la HU-__ usando las variables {{baseUrl}} y {{token}}. Incluye el caso correcto y los casos de error indicados en la tarjeta, con sus pruebas (assert/tests) del código HTTP esperado.
```

## 5. Reglas para no romper lo que ya funciona
1. **Una tarea por chat.** Si la conversación se llena de errores, abre otra con el error y el archivo, no insistas.
2. **El contrato manda.** Si hace falta cambiar un endpoint, primero se edita `02-API.md`, se anota en `06-ESTADO.md` y después se corrigen backend y frontend con tareas separadas.
3. **Archivos completos.** Rechaza respuestas con "aquí va el resto igual".
4. **Compilar y probar antes de seguir.** Después de cada tarea: compilar, Bruno, navegador, commit.
5. **No mezclar módulos.** Si la IA modifica autenticación mientras haces pedidos, descarta ese cambio.
6. **Guarda cada avance en Git.** Si una IA rompe algo, vuelves con `git restore` o `git checkout <commit> -- archivo`.
7. **Secretos.** Nunca pegues contraseñas, tokens ni claves reales de Twilio en un chat; usa `CAMBIAR`.
8. **Pregunta por dependencias.** Si una HU depende de otra que no está hecha (por ejemplo Postfix), la IA debe dejar el comportamiento correcto (log FALLIDO), nunca un simulacro.
