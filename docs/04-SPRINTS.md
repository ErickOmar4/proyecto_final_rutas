# 04 · SPRINTS (4 días + preparación)

Regla de oro: **cada día termina con todo lo del día funcionando, desplegado en el VPS, con commits y con las tarjetas movidas en GitHub Projects.** Nada de "casi terminado".
Dentro de cada día, el orden de las historias es el orden de trabajo. Cada historia se hace con **una conversación nueva de IA** (ver `07-PROMPTS-IA.md`).

## Ritual de cada tarea (10 a 15 minutos de control)
1. Mover la tarjeta a **In Progress**.
2. Abrir un chat nuevo y pegar: prompt maestro + `00-CONTEXTO.md` + sección de `02-API.md` que toca + la tarjeta de la HU + solo los archivos a modificar.
3. Copiar los archivos que entregue la IA (completos, con su ruta).
4. Compilar: `mvn -q compile` y/o `npm run build`. Si falla dos veces, chat nuevo con el error y el archivo.
5. Probar con Bruno la petición de la HU y revisar la pantalla en el navegador.
6. `git add . && git commit -m "feat(modulo): descripcion" && git push`.
7. Mover la tarjeta a **Review**, revisar los criterios de aceptación uno por uno y pasar a **Done**.
8. Actualizar `06-ESTADO.md` (qué quedó, qué falló, decisiones nuevas).

## Cierre de cada día (30 minutos)
- `deploy/deploy.ps1` y comprobar en `https://rutaoax.devs8.tech`.
- Correr toda la colección de Bruno contra producción: nada en rojo salvo los casos de error esperados.
- Commit con el estado del día y `06-ESTADO.md` actualizado.
- Si algo del día no se terminó, pasa al día siguiente como **primera** tarea; no se acumulan dos días de atraso.

---
## DÍA 0 · Preparación (2 a 3 horas)
| Orden | Tarea | Historia | Resultado |
|---|---|---|---|
| 1 | Crear repo, carpetas, `.gitignore`, copiar `docs/`, `deploy/`, `diseno/` y el SQL | HU-00 | Repo público listo |
| 2 | Crear el tablero de GitHub Projects con las 4 columnas y una tarjeta por HU | HU-00 | Tablero público |
| 3 | Crear los registros DNS (A, MX, SPF, DMARC) | HU-00 | `dig` los resuelve |
| 4 | Preparar el VPS (Java, MySQL 8, Nginx, Certbot, usuario, carpetas) | HU-00 | `05-DESPLIEGUE.md` secciones 2 a 5 |
| 5 | Instalar Postfix y OpenDKIM y publicar la clave DKIM (la propagación tarda) | HU-26 | Registro DKIM publicado |
| 6 | Crear cuenta de Twilio, verificar tu número y unirte al sandbox de WhatsApp | HU-00 | Número verificado |
| 7 | Reparto entre los dos integrantes (quién hace backend de qué módulo, quién frontend) | | Tarjetas con responsable |

## DÍA 1 · Base, autenticación y primer despliegue
Meta: entrar con los 4 roles en `https://rutaoax.devs8.tech` y ver su pantalla vacía.
| Bloque | Historias | Qué se entrega |
|---|---|---|
| Mañana | HU-01, HU-02 | Backend arranca, Flyway crea las 12 tablas, seed cargado, errores JSON |
| Mediodía | HU-03, HU-04 | Registro y login JWT (backend + pantallas), primera colección Bruno con token |
| Tarde | HU-05, HU-06, HU-08 | Logout con revocación, recuperación de contraseña, seguridad por rol |
| Noche | HU-07, HU-09 | Rutas protegidas, navbar, perfil y **primer despliegue con HTTPS** |

Criterio de cierre: login de `admin@rutaoax.com`, `negocio1@`, `repartidor1@`, `independiente1@` y `cliente1@rutaoax.com` en producción, cada uno redirigido a su sección; token revocado devuelve 401; Bruno verde.
Si te atrasas: deja HU-06 con el correo en FALLIDO (se cierra el día 3 con Postfix) pero no recortes HU-08 ni HU-09.

## DÍA 2 · Negocio
Meta: un negocio crea pedidos, los asigna a un repartidor con vehículo y ve su panel.
| Bloque | Historias | Qué se entrega |
|---|---|---|
| Mañana | HU-10, HU-11 | Vehículos y equipo (backend + pantallas) |
| Tarde | HU-12, HU-13 | Pedidos con paginación/filtros del servidor y asignación con reglas |
| Noche | HU-14, HU-15 | Panel del negocio, publicar y reasignar |

Criterio de cierre: como `negocio1@`, crear un pedido, asignarlo a `repartidor1@` con un vehículo y ver el cupo bajar; probar que `negocio2@` no ve ni toca nada de `negocio1@`.
Si te atrasas: recorta HU-15 y la tarjeta "Requiere atención" de HU-14 (queda solo con KPIs).

## DÍA 3 · Repartidores, cliente, mensajes y comunicaciones
Meta: el flujo completo de punta a punta con SMS y WhatsApp reales.
| Bloque | Historias | Qué se entrega |
|---|---|---|
| Mañana | HU-25, HU-26 | Servicio de notificaciones y correo real con Postfix (probar con el endpoint de prueba) |
| Mediodía | HU-16, HU-17, HU-18 | Repartidor de empresa: ruta, iniciar (SMS), entregar (WhatsApp) |
| Tarde | HU-19, HU-20 | Independiente: disponibles, tomar y ruta en mapa |
| Noche | HU-21, HU-23, HU-24 | Seguimiento del cliente y mensajes de los tres roles |

Criterio de cierre: flujo completo en producción: el negocio asigna → el repartidor inicia (llega SMS a tu celular) → marca entregado (llega WhatsApp) → el cliente ve el seguimiento actualizado → escribe un mensaje y lo recibe quien corresponde.
Si te atrasas: HU-22 y el Historial del repartidor no se hacen. Nunca se recortan HU-17, HU-18 ni HU-25 (valen puntos de comunicación).

## DÍA 4 · Administración, calidad y entrega
Meta: sistema cerrado, documentado y ensayado. **Después de las 14:00 no se agrega ninguna función nueva.**
| Bloque | Historias | Qué se entrega |
|---|---|---|
| Mañana | HU-27, HU-28, HU-29 | Pantallas de administrador y matriz de permisos con Bruno |
| Mediodía | HU-30, HU-31 | Validaciones y estados de interfaz, colección Bruno completa |
| Tarde | HU-32, HU-33 | README + ER + respaldo SQL; Figma, logo y paleta justificada |
| Noche | HU-34 | Despliegue final y ensayo completo de la demo con cada rol |

Paralelo recomendado: mientras una IA genera código, el otro integrante avanza Figma (HU-33) y el README (HU-32).
Criterio de cierre: `08-RUBRICA-Y-DEMO.md` completo con todas las casillas marcadas.

## Reparto entre dos integrantes
La rúbrica evalúa a cada integrante y revisa commits de ambos. Propuesta:
- **Integrante A:** backend de auth, negocio, vehículos, pedidos y asignación; Bruno; despliegue.
- **Integrante B:** frontend (layout, auth, pantallas por rol), mensajes, notificaciones; Figma; README.
- Cada uno hace commits propios todos los días y, al terminar, le explica su módulo al otro: en la demo se pregunta por separado.
