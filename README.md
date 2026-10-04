# Homelab

Mi laboratorio personal de sistemas y redes. Lo monto para aprender administración de sistemas y soporte IT con la práctica, y cada proyecto queda documentado aquí: qué hice, por qué, qué se rompió y qué aprendí.

## Sobre mí

Rubén. Estoy cambiando de carrera hacia IT (soporte N1 → administración de sistemas → DevOps). Estoy haciendo el certificado Google IT Support y voy a empezar el ciclo SMX (Sistemas Microinformáticos y Redes).

## Estado del laboratorio

| Elemento | Estado |
|---|---|
| Portátil de estudio con Fedora 44 (GNOME) | En uso |
| Servidor 24/7 (HP Compaq 6005 Pro reciclado) | En marcha (v1.3): Ubuntu Server por SSH, NAS Samba, firewall, SSH con llave y fail2ban. Docker con Portainer. Faltan copias y acceso remoto |
| Segundo PC de oficina (HP Compaq 8000 Elite) | Aparcado: error de memoria (5 pitidos) |
| Mini PC para virtualización (Proxmox, Active Directory) | Aplazado para más adelante |

## Proyectos

Cada proyecto tiene su carpeta y su README. Solo se documenta lo que he hecho de verdad.

| Carpeta | Proyecto | Estado |
|---|---|---|
| [01-debian-base](01-debian-base/) | Debian sin escritorio como base del servidor | Pendiente |
| [02-pi-hole](02-pi-hole/) | Pi-hole: bloqueo de anuncios y DNS local | En curso: instalado y probado, falta que lo use toda la red |
| [03-ssh-y-seguridad](03-ssh-y-seguridad/) | Hardening: ufw, SSH solo con llave, fail2ban, logs y nmap | Terminado (nivel básico) |
| [04-docker](04-docker/) | Docker y servicios (Portainer, Uptime Kuma…) | En curso |
| [05-redes-basicas](05-redes-basicas/) | IP, gateway y NAT comprobados en mi portátil | Terminado |
| [06-servidor-hp-6005-pro](06-servidor-hp-6005-pro/) | Servidor 24/7 con un HP Compaq 6005 Pro reciclado: diagnóstico, BIOS, discos e instalación | En curso |
| [07-nas-samba](07-nas-samba/) | NAS con Samba: usuarios, grupos, permisos y carpetas compartidas | Terminado |

## Cómo documento cada proyecto

Uso la plantilla de [`plantillas/README-proyecto.md`](plantillas/README-proyecto.md):

1. **Qué es** y por qué lo hago.
2. **Cómo lo hice**, paso a paso.
3. **Qué se rompió** y cómo lo arreglé.
4. **Resultado.**
5. **Qué aprendí.**

## Norma

No se sube nada sensible: contraseñas, claves, tokens ni IPs privadas.
