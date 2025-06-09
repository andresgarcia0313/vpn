# 🛡️ VPN con WireGuard + WG-Easy

Este proyecto permite desplegar rápidamente una VPN basada en **WireGuard** usando la interfaz web de **WG-Easy**, todo a través de Docker. Ideal para tener una VPN segura, autogestionada y fácil de usar, incluso para quienes no son expertos en redes.

---

## 🚀 ¿Qué hace este stack?

- Levanta un contenedor con WG-Easy para administrar tu VPN vía interfaz web.
- Usa WireGuard como backend, proporcionando conexiones rápidas y seguras.
- Accesible desde cualquier navegador a través del puerto TCP configurado.
- Configurado para reiniciarse automáticamente si el sistema se reinicia.

---

## 🧱 Requisitos

- Docker y Docker Compose instalados
- Un dominio en GoDaddy o similar
- Un router con acceso a la configuración de puertos
- IP pública estática (o dinámica con DDNS)
- Ubuntu/Debian u otra distro compatible

---

## 🌐 Configuración en GoDaddy (Dominio personalizado)

1. **Accede a tu cuenta GoDaddy**.
2. Ve a **Mis productos > DNS** del dominio que usarás (ej. `vpn.dominio.com`).
3. En la sección **Registros DNS**, agrega o edita un registro tipo **A**:
   - **Nombre**: `vpn` (o el subdominio que prefieras)
   - **Tipo**: A
   - **Valor**: tu dirección IP pública (puedes verla en [https://whatismyipaddress.com](https://whatismyipaddress.com))
   - **TTL**: 600 segundos (o automático)

📌 Espera unos minutos a que propaguen los cambios (hasta 30 minutos en promedio).

---

## 🔐 Hash de contraseña

Para acceder al panel web, necesitas un **hash de contraseña encriptado**. Puedes generarlo con:

```bash
sudo apt install apache2-utils
htpasswd -nbB admin tu_contraseña
````

🔁 Si tu contraseña contiene `$`, reemplázalo por `$$` al definir la variable en tu entorno o `.env`.

---

## 📦 Estructura `docker-compose.yml`

```yaml
services:
  wg-easy:
    image: ghcr.io/wg-easy/wg-easy:latest
    container_name: wg-easy
    cap_add:
      - NET_ADMIN
      - SYS_MODULE
    environment:
      - WG_HOST=${WG_HOST}     # Dominio/IP pública
      - PASSWORD_HASH=${PASSWORD_HASH}        # Hash generado
      - PORT=51821                             # Puerto interfaz web
      - WG_PORT=51820                          # Puerto VPN
      - TZ=America/Bogota                      # Zona horaria
    volumes:
      - ./wireguard-data:/etc/wireguard
    ports:
      - "51820:51820/udp"
      - "51821:51821/tcp"
    sysctls:
      - net.ipv4.conf.all.src_valid_mark=1
      - net.ipv4.ip_forward=1
    restart: unless-stopped
```

---

## 📶 Configuración del router del ISP

Para que los dispositivos remotos puedan conectarse a tu servidor VPN, debes redirigir puertos desde tu router:

1. Accede a tu router desde el navegador (normalmente `192.168.1.1` o `192.168.0.1`).
2. Busca la sección **Port Forwarding / NAT / Reenvío de puertos**.
3. Agrega dos reglas:

| Nombre     | Puerto externo | Puerto interno | Protocolo | IP del servidor |
| ---------- | -------------- | -------------- | --------- | --------------- |
| WG-Easy UI | 51821          | 51821          | TCP       | `192.168.x.x`   |
| WireGuard  | 51820          | 51820          | UDP       | `192.168.x.x`   |

> 🔁 Asegúrate de reemplazar `192.168.x.x` por la IP local de tu servidor Docker.

4. Guarda los cambios y reinicia el router si es necesario.

📌 Si tu IP pública cambia con el tiempo (IP dinámica), puedes usar un servicio DDNS como [No-IP](https://www.noip.com) o [DuckDNS](https://www.duckdns.org) y configurarlo como `WG_HOST`.

---

## 🧪 Cómo usar

1. Crea un archivo `.env` en el mismo directorio con tu hash:

```
PASSWORD_HASH=tu_hash_generado_aquí
```

2. Inicia el contenedor:

```bash
docker-compose up -d
```

3. Accede al panel de WG-Easy desde tu navegador:

```
http://vpn.domino.com:51821
```

---

## 📁 Datos persistentes

Los archivos de configuración y claves se almacenan en `./wireguard-data`. Esto permite conservar la configuración aunque reinicies el contenedor.

---

## 🌍 Notas adicionales

* Asegúrate de que los puertos 51820 (UDP) y 51821 (TCP) estén abiertos en el firewall del sistema y del router.
* Puedes usar Cloudflare para mejorar seguridad y velocidad DNS si deseas usar un proxy DNS.

---

## ❤️ Créditos

Configurado y documentado por [Andrés Eduardo García Márquez](https://ingeniumcodex.com) andresgarcia0313@gmail.com
Imagen WG-Easy usada

---

## ☕ ¿Te fue útil?

Si este proyecto te ahorró tiempo o dolores de cabeza, considera compartirlo o contribuir. ¡Cada gesto cuenta! 🙌
