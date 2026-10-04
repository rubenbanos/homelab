# NAS con Samba en el servidor

**Estado:** terminado (v1)
**Fecha:** 2026-10-03
**Equipo:** servidor HP Compaq 6005 Pro con Ubuntu Server 26.04 (ver [`06-servidor-hp-6005-pro`](../06-servidor-hp-6005-pro/))

## Qué es y por qué lo hago

Un disco de 1 TB en red para que mi hermano y yo guardemos archivos, cada uno en su carpeta privada y una más compartida, sin depender de la nube. También es práctica real de usuarios, grupos y permisos en Linux, que es lo que se pregunta en soporte N1.

## Cómo lo hice

1. **Disco de datos** formateado en ext4 con etiqueta `nas`, montado en `/srv/nas` desde `/etc/fstab` con `nofail` (el servidor arranca aunque el disco falle).
2. **Usuarios y grupo:** un usuario por persona (sin carpeta personal ni acceso por SSH) y un grupo `nas` con los dos.
3. **Carpetas y permisos:**

| Carpeta | Propietario | Permisos | Quién entra |
|---|---|---|---|
| `Ruben` | yo | `700` | solo yo |
| `Hermano` | mi hermano | `700` | solo él |
| `Comun` | yo, grupo `nas` | `2770` (con setgid) | los dos |

4. **Samba:** una contraseña de Samba aparte para cada usuario (`smbpasswd -a`) y un bloque por carpeta al final de `/etc/samba/smb.conf`:

```ini
[Comun]
   path = /srv/nas/Comun
   valid users = @nas
   read only = no
   browseable = yes
   create mask = 0660
   directory mask = 2770
   force group = nas
```

5. **Validar y reiniciar:** `testparm` comprueba la sintaxis y `sudo systemctl restart smbd` aplica el cambio.
6. **Probar desde el cliente:** en Windows con «Conectar unidad de red» y en el Mac de mi hermano con `Cmd+K` → `smb://<ip-del-servidor>`.
7. **Pruebas de resistencia:** reinicio del servidor y corte de luz; el disco se monta solo y Samba arranca solo.

## Qué se rompió

- **Problema:** al conectar la unidad de red desde Windows, la contraseña no valía.
  - **Causa:** Samba guarda sus contraseñas aparte de las de Linux, y Windows puede recordar credenciales antiguas.
  - **Solución:** crear la contraseña con `smbpasswd` y conectar con «otras credenciales».
- **Problema:** las unidades de red aparecen con una X roja al arrancar Windows.
  - **Causa:** Windows las mapeó cuando el servidor aún estaba apagado.
  - **Solución:** doble clic para reconectar; no es un fallo de Samba.
- **Problema:** olvidé la contraseña de Samba.
  - **Causa:** son tres claves distintas (la frase de mi llave SSH, la contraseña de Linux y la de Samba) y las confundía.
  - **Solución:** `sudo smbpasswd <usuario>` la cambia sin tocar las otras dos.

## Resultado

Mi carpeta y la común funcionan desde Windows, y la suya y la común desde su Mac. Un reinicio y un corte de luz no rompen nada.

## Qué aprendí

- **Samba tiene su propia lista de contraseñas**, separada de la del sistema.
- **Permisos Linux en la práctica:** `700` para lo privado y `2770` con setgid para una carpeta de grupo, de modo que lo que cree uno lo pueda editar el otro.
- **Probar desde el cliente, no solo en el servidor:** un firewall o un permiso solo se ven bien desde fuera.
- **Nunca abrir puertos del router para Samba:** es un servicio para la red local.
- **Un disco no es una copia de seguridad:** falta añadir `rsync` + `cron` a otro destino.

## Recursos

- Documentación de Samba: https://www.samba.org/samba/docs/
