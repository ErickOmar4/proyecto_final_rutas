# 05 · DESPLIEGUE EN EL VPS

> **Atajo (Parte 4):** `deploy/preparar-servidor.sh` automatiza las secciones 2 a 6 (paquetes, firewall, base de datos, carpetas, configuración, Nginx, systemd y, si el DNS ya resuelve, el certificado HTTPS). Cada publicación posterior se hace con `deploy/deploy.ps1`. Las secciones 7 (Postfix, SPF y DKIM) y 8 (Twilio) siguen siendo manuales y son del Día 3. Este documento queda como referencia detallada de lo que hace el script.

Datos: IP `179.197.84.34` · dominio `devs8.tech` · sitio `rutaoax.devs8.tech` · backend en el puerto interno `9250` · carpeta de la app `/opt/rutaoax` · frontend en `/var/www/rutaoax`.
Se asume Ubuntu 22.04/24.04 y acceso como `root` por SSH. Si el servidor es otro sistema, avisa a la IA con el resultado de `cat /etc/os-release`.
Cada paso indica qué deberías ver. Si no ves eso, detente y resuelve antes de seguir.

## 1. DNS (en el panel donde compraste devs8.tech)
| Tipo | Nombre | Valor |
|---|---|---|
| A | rutaoax | 179.197.84.34 |
| A | mail | 179.197.84.34 |
| MX | @ | mail.devs8.tech (prioridad 10) |
| TXT | @ | `v=spf1 ip4:179.197.84.34 ~all` |
| TXT | _dmarc | `v=DMARC1; p=none; rua=mailto:admin@devs8.tech` |
| TXT | mail._domainkey | (se obtiene en el paso 7) |

Verifica desde tu PC: `nslookup rutaoax.devs8.tech` → debe mostrar 179.197.84.34.
Pide también a tu proveedor del VPS que configure el **DNS inverso (PTR)** de la IP como `mail.devs8.tech`; ayuda mucho a no caer en spam.
Si ya usas `devs8.tech` para otra cosa, **no** cambies el registro MX/SPF existente: pregunta primero.

## 2. Paquetes base
```bash
ssh root@179.197.84.34
apt update && apt install -y openjdk-21-jre-headless mysql-server nginx certbot python3-certbot-nginx ufw
java -version        # debe decir 21
mysql --version      # debe decir "MySQL", NO "MariaDB"
```
Si `mysql --version` dice MariaDB, **no sirve para la rúbrica**: desinstálalo e instala MySQL 8 desde el repositorio oficial (`dev.mysql.com/downloads/repo/apt`).

## 3. Firewall
```bash
ufw allow OpenSSH && ufw allow 'Nginx Full' && ufw allow 25/tcp && ufw --force enable
ufw status           # OpenSSH, Nginx Full y 25 en ALLOW
```
El puerto 9250 y el 3306 **no** se abren: solo se usan en el propio servidor.

## 4. Base de datos
```bash
mysql
```
```sql
CREATE DATABASE rutaoax CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
CREATE USER 'rutaoax_app'@'localhost' IDENTIFIED BY 'PON_UNA_CLAVE_LARGA';
GRANT ALL PRIVILEGES ON rutaoax.* TO 'rutaoax_app'@'localhost';
FLUSH PRIVILEGES;
EXIT;
```
Prueba: `mysql -u rutaoax_app -p rutaoax -e "SELECT 1;"` → muestra una tabla con 1.

## 5. Carpetas, usuario y configuración
```bash
useradd -r -s /usr/sbin/nologin rutaoax
mkdir -p /opt/rutaoax/backend /opt/rutaoax/config /var/www/rutaoax
nano /opt/rutaoax/config/application.properties
```
Pega el contenido de `application.properties.example` y reemplaza todos los `CAMBIAR`. Para producción:
- `spring.datasource.password` = la clave del paso 4
- `app.jwt.secret` = una cadena aleatoria de 40+ caracteres (`openssl rand -base64 48`)
- `app.cors.allowed-origins=https://rutaoax.devs8.tech` y `app.frontend-url=https://rutaoax.devs8.tech`
```bash
chown -R rutaoax:rutaoax /opt/rutaoax
chmod 600 /opt/rutaoax/config/application.properties
```

## 6. Nginx, systemd y HTTPS
Desde tu PC (en la carpeta del repo) sube los archivos de `deploy/`:
```powershell
scp deploy/nginx-rutaoax.conf root@179.197.84.34:/etc/nginx/sites-available/rutaoax
scp deploy/rutaoax.service root@179.197.84.34:/etc/systemd/system/rutaoax.service
```
En el servidor:
```bash
ln -s /etc/nginx/sites-available/rutaoax /etc/nginx/sites-enabled/rutaoax
nginx -t                                  # "syntax is ok"
systemctl reload nginx
systemctl daemon-reload && systemctl enable rutaoax
certbot --nginx -d rutaoax.devs8.tech     # elige redirigir HTTP a HTTPS
```
Debes ver "Congratulations!". Comprueba `https://rutaoax.devs8.tech` (candado válido). La renovación es automática (`systemctl status certbot.timer`).

## 7. Postfix + SPF + DKIM (correo real)
```bash
apt install -y postfix mailutils opendkim opendkim-tools
```
En el asistente de Postfix elige **Internet Site** y como nombre del sistema `devs8.tech`.
```bash
postconf -e "myhostname = mail.devs8.tech"
postconf -e "mydomain = devs8.tech"
postconf -e "myorigin = /etc/mailname"
postconf -e "inet_interfaces = loopback-only"
postconf -e "inet_protocols = ipv4"
postconf -e "milter_default_action = accept"
postconf -e "milter_protocol = 6"
postconf -e "smtpd_milters = inet:localhost:8891"
postconf -e "non_smtpd_milters = inet:localhost:8891"
```
`loopback-only` significa que solo la app del propio servidor puede enviar (no es un relay abierto).

Clave DKIM:
```bash
mkdir -p /etc/opendkim/keys/devs8.tech
opendkim-genkey -b 2048 -d devs8.tech -D /etc/opendkim/keys/devs8.tech -s mail
chown -R opendkim:opendkim /etc/opendkim
cat > /etc/opendkim.conf <<'CONF'
Syslog yes
UMask 002
Socket inet:8891@localhost
Domain devs8.tech
KeyFile /etc/opendkim/keys/devs8.tech/mail.private
Selector mail
Canonicalization relaxed/simple
CONF
systemctl restart opendkim postfix
cat /etc/opendkim/keys/devs8.tech/mail.txt
```
Copia lo que está entre comillas de `mail.txt` (junta las líneas) como valor del registro TXT `mail._domainkey` del paso 1. Espera unos minutos y verifica: `dig TXT mail._domainkey.devs8.tech +short`.

Prueba de envío (cambia el destino por tu Gmail):
```bash
echo "Prueba RutaOax" | mail -s "Prueba" -a "From: no-reply@devs8.tech" tucorreo@gmail.com
tail -n 20 /var/log/mail.log      # debe decir status=sent
```
Revisa que llegue a **Recibidos**; en Gmail → ⋮ → "Mostrar original" deben aparecer `SPF: PASS` y `DKIM: PASS`. También puedes mandar uno a la dirección que te dé `mail-tester.com`.
Si queda en `Connection timed out` hacia los servidores de Google, tu proveedor bloquea el puerto 25 de salida: abre un ticket para que lo habiliten (es la causa más común).

## 8. Twilio
1. En la consola de Twilio copia `Account SID` y `Auth Token`.
2. **SMS:** compra o usa el número de prueba; en cuenta trial solo envía a números **verificados** (verifica el tuyo y el del integrante). Pon ese número en `app.twilio.sms-from`.
3. **WhatsApp:** en "Messaging → Try it out → Send a WhatsApp message" envía desde tu WhatsApp el código `join xxxx-xxxx` al número del sandbox. Cada número destino debe hacerlo (dura 72 h; repítelo antes de la demo). El remitente ya está en `app.twilio.whatsapp-from`.
4. Pon esos valores en `/opt/rutaoax/config/application.properties`, reinicia con `systemctl restart rutaoax` y prueba con `POST /api/admin/notificaciones/prueba`.

## 9. Publicar cada vez (desde tu PC, raíz del repo)
```powershell
./deploy/deploy.ps1
```
Debe terminar imprimiendo `active`. Ver registros del backend: `ssh root@179.197.84.34 "journalctl -u rutaoax -n 50 --no-pager"`.

## 10. Comprobaciones finales
| Prueba | Esperado |
|---|---|
| `https://rutaoax.devs8.tech` | login con candado válido |
| `curl -i https://rutaoax.devs8.tech/api/zonas` | 401 en JSON (sin token) |
| `systemctl status rutaoax` | active (running) |
| Reiniciar el servidor (`reboot`) | la app vuelve sola |
| `mysql ... -e "SELECT COUNT(*) FROM flyway_schema_history;"` | 1 o más |

## Problemas frecuentes
| Síntoma | Causa probable |
|---|---|
| 502 Bad Gateway | el JAR no arrancó: `journalctl -u rutaoax -n 80` |
| Pantalla en blanco al recargar una ruta | falta `try_files $uri /index.html;` en Nginx |
| CORS en el navegador | `app.cors.allowed-origins` no coincide con la URL |
| `Access denied for user` | usuario o clave de MySQL distintos en `application.properties` |
| Correo en spam | falta PTR, SPF o DKIM; revisa el "Mostrar original" |
