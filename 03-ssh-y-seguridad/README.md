# Acceso por SSH con claves y protección básica (hardening)

**Estado:** terminado (nivel básico)
**Fecha:** 2026-10-03 y 2026-10-04
**Equipo:** servidor HP Compaq 6005 Pro con Ubuntu Server 26.04, manejado desde mi torre con Windows (PowerShell)

## Qué es y por qué lo hago

Asegurar mi propio servidor aplicando, una a una, las medidas básicas de defensa, y entender para qué sirve cada una. Me interesa saber si la ciberseguridad defensiva me engancha, y un servidor propio es el mejor sitio para practicarla.

## Cómo lo hice

La idea es la **defensa en profundidad**: varias capas, de modo que si una falla, las demás siguen protegiendo.

1. **Actualizaciones automáticas de seguridad** (`unattended-upgrades`, ya activo por defecto): comprobar `/etc/apt/apt.conf.d/20auto-upgrades`. Los parches de kernel solo se aplican al reiniciar.
2. **Firewall (`ufw`):** primero las reglas y solo después `enable`. Política de denegar todo lo entrante y permitir solo, desde mi red local, SSH, Samba, DNS y el panel web.

```bash
sudo ufw default deny incoming
sudo ufw default allow outgoing
sudo ufw allow from <mi-red>/24 to any port 22 proto tcp
# ...una regla por servicio...
sudo ufw enable
sudo ufw status
```

3. **SSH solo con clave:** par ed25519 generado en mi torre **con frase de contraseña**, clave pública añadida a `~/.ssh/authorized_keys` del servidor, contraseñas desactivadas en un archivo propio:

```bash
echo "PasswordAuthentication no" | sudo tee /etc/ssh/sshd_config.d/01-hardening.conf
sudo sshd -t                                   # sin salida = configuración válida
sudo sshd -T | grep -i passwordauthentication  # debe decir "no"
sudo systemctl reload ssh
```

   Antes de desactivar las contraseñas probé la llave **en una sesión nueva**, sin cerrar la anterior. Y comprobé que sin llave no entra: `Permission denied (publickey)`.
4. **fail2ban:** instalado con `sudo apt install fail2ban`. La jail `sshd` viene activa y lee el journal de systemd; por defecto, 5 fallos en 10 minutos → baneo de 10 minutos. Se comprueba con `sudo fail2ban-client status sshd`.
5. **Leer los logs:** `sudo journalctl -u ssh -n 20 --no-pager`. Solo aparecían mis propios accesos.
6. **Escanear mi propio servidor con nmap** desde otro equipo de la red: `nmap <ip-del-servidor>`.

## Qué se rompió

- **Problema:** `sshd -t` dio error al comprobar mi configuración.
  - **Causa:** una errata en la directiva (`PasswordAutentication`, sin la «h»).
  - **Solución:** corregirla. Gracias a validar con `sshd -t` **antes** de recargar, no me quedé fuera.
- **Problema:** el primer `ssh` dio `Connection timed out`.
  - **Causa:** el servidor estaba apagado.
  - **Solución:** encenderlo. Los tres errores típicos son distintos: *timed out* = no responde (apagado o firewall), *refused* = responde pero SSH no escucha, *Permission denied (publickey)* = la llave o el usuario no valen.
- **Problema:** `sudo apt install fail2ban` falló a la primera.
  - **Causa:** olvidé el `sudo`.
  - **Solución:** repetirlo con `sudo`.
- **Problema:** en los logs, mi torre aparecía con una IP distinta a la del día anterior.
  - **Causa:** el DHCP del router le asignó otra.
  - **Solución:** ninguna; el firewall permite toda mi red local. Pero me enseñó a distinguir lo normal de lo sospechoso.

## Resultado

- Escaneo desde la red local: **solo 4 puertos abiertos** (SSH, DNS, web de Pi-hole y Samba) y los otros 996 aparecen como *filtered*. Coincide exactamente con las 4 reglas del firewall.
- El servidor solo acepta mi llave privada: saber el usuario y la contraseña no basta para entrar por SSH.

## Qué aprendí

- **Las capas se complementan:** router → firewall → solo llave → fail2ban. Ninguna es perfecta sola.
- **fail2ban solo ve fallos.** Un `Failed` en el log es ruido de internet; un **`Accepted` desde una IP que no es de mi red es la alarma**, y fail2ban no la vería.
- **`filtered` es mejor que `closed`:** el firewall descarta sin contestar, el atacante no sabe si hay algo y pierde tiempo (mi escaneo tardó unos 3 minutos).
- **Reglas primero, `enable` después:** si activas `ufw` antes de permitir el puerto 22 te quedas fuera (la salida es monitor y teclado en el propio servidor).
- **Validar antes de aplicar** (`sshd -t`) y probar la llave en una sesión nueva antes de cerrar la que funciona.
- **Para entrar desde fuera no se abre el puerto 22 en el router:** se usa una VPN (Tailscale, pendiente).
- **Docker se salta `ufw`** al publicar puertos: hay que tenerlo en cuenta cuando llegue.
- **Solo se escanean máquinas propias.**

## Recursos

- Manual de `ufw`: https://help.ubuntu.com/community/UFW
- fail2ban: https://github.com/fail2ban/fail2ban
- nmap: https://nmap.org/book/man.html
