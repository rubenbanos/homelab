# Servidor 24/7 con un HP Compaq 6005 Pro reciclado

**Estado:** en curso (v1.3: arranca, NAS Samba y hardening básico funcionando; faltan Docker, copias de seguridad y acceso remoto)
**Fecha:** 2026-09-27 (inicio) · actualizado 2026-10-04
**Equipo:** HP Compaq 6005 Pro SFF (2009-2010), rescatado de un trastero

## Qué es y por qué lo hago

Quiero un servidor encendido las 24 horas, separado de mi torre (juegos) y de mi portátil (estudio), para practicar lo que se pide en soporte N1 y en el ciclo SMX: Linux sin escritorio por SSH, un NAS con Samba, Pi-hole y Docker. En vez de comprar un mini PC, aproveché dos PCs de oficina que tenía guardados y comprobé cuál servía.

## El equipo

| Dato | Valor |
|---|---|
| CPU | AMD Phenom II X2 B55, 2 núcleos a 3,0 GHz (soporta AMD-V). Pendiente: cambiarla por una X4 B97 |
| RAM | 4 GB DDR3 (4 × 1 GB), ampliable a 16 GB |
| Placa | HP con chipset AMD 785G, BIOS 786G6 (actualizada a v01.17) |
| Sistema | SSD Samsung de 256 GB con Ubuntu Server 26.04 LTS |
| Datos | HDD Seagate de 1 TB (NAS) |
| Fuente | 240 W |
| Consumo estimado en reposo | 35-50 W |

Da de sobra para Ubuntu Server + Samba + Pi-hole + Docker ligero. Se queda corto para Proxmox con máquinas virtuales Windows, y eso lo dejo para un equipo más potente más adelante.

## Cómo lo hice

1. **Identificar los dos equipos** por la etiqueta: no eran Dell OptiPlex como creía, sino un HP Compaq 6005 Pro y un HP Compaq 8000 Elite.
2. **Probar cada uno por descarte:** cable y enchufe, monitor, teclado, memoria.
3. **Resetear la CMOS** del 6005 Pro con su interruptor SW1 (PC desenchufado, mantenido unos 10 segundos) y **cambiar la pila** por una CR2032 nueva.
4. **Configurar la BIOS** (F10 en el arranque):

| Menú | Opción | Valor |
|---|---|---|
| Archivo | Hora y fecha | ajustadas |
| Seguridad | Tecnología de virtualización | Habilitada (AMD-V) |
| Avanzado → Opciones de arranque | Después de pérdida de energía | Encendido |
| Almacenamiento | Emulación SATA | AHCI |

   Guardar con **Archivo → Guardar cambios y salir** (en esta BIOS, F10 dentro de un submenú solo acepta la opción y no guarda).
5. **Retirar el lector de DVD** para liberar su jaula, donde va el SSD (de momento suelto: falta sujetarlo).
6. **Actualizar la BIOS a v01.17** desde un pendrive FAT32 (SoftPaq de HP), con la CPU antigua puesta, antes de montar la nueva.
7. **Instalar Ubuntu Server** con OpenSSH, verificando antes el SHA256 de la ISO.
8. **IP fija con netplan:** el router no me deja reservar la IP (no tengo la contraseña de administrador), así que la fijé en el propio servidor y la probé con `netplan try`, que revierte solo a los 120 s si algo falla.
9. **Formatear y montar el disco de datos** (`parted`, `mkfs.ext4` con etiqueta, línea en `/etc/fstab` con `nofail`) y comprobar su estado con `smartctl` antes de guardar nada.
10. **NAS con Samba y hardening:** documentados en [`07-nas-samba`](../07-nas-samba/) y [`03-ssh-y-seguridad`](../03-ssh-y-seguridad/).

## Qué se rompió

- **Problema:** el segundo PC (8000 Elite) no arrancaba: cinco pitidos y el LED rojo parpadeando cinco veces.
  - **Causa:** en la tabla de códigos de HP, cinco pitidos significan error de memoria.
  - **Solución:** de momento aparcado. Prueba pendiente: montarle un módulo del 6005 Pro.
- **Problema:** mi teclado mecánico de juegos no respondía en la BIOS.
  - **Causa:** las BIOS antiguas no reconocen bien los teclados gaming (protocolo distinto al de los teclados sencillos).
  - **Solución:** usar un teclado básico.
- **Problema:** tras resetear la CMOS apareció el aviso 162.
  - **Causa:** el reset borra la configuración y la hora guardadas.
  - **Solución:** volver a configurar la BIOS y guardar.
- **Problema:** la primera instalación de Ubuntu falló.
  - **Causa:** la ISO se había descargado corrupta al pendrive.
  - **Solución:** comprobar el SHA256 de la ISO contra el oficial y volver a descargarla. Desde entonces verifico siempre el hash, y también lo que queda grabado en el pendrive.
- **Problema:** Ubuntu quedó instalado pero el servidor no arrancaba (cursor parpadeando tras «Attempting Boot From Hard Drive»).
  - **Causa:** con dos discos, esta BIOS intenta arrancar del primero (SATA0) y ahí estaba el disco de datos, vacío. La BIOS solo deja elegir la categoría «Disco duro», no cuál.
  - **Solución:** intercambiar los cables de datos para que el SSD del sistema quede en SATA0.
- **Problema:** al editar `/etc/fstab` con nano se coló un carácter extra en el nombre del archivo y nano preguntó si guardar con otro nombre.
  - **Causa:** una tecla pulsada de más al guardar.
  - **Solución:** cancelar, borrar el carácter y guardar con el nombre correcto. Moraleja: leer bien lo que pregunta nano antes de pulsar Intro.

## Resultado

El servidor arranca solo tras un corte de luz (probado), se maneja entero por SSH sin monitor ni teclado, tiene IP fija y comparte el disco de 1 TB por Samba con la red de casa.

## Qué aprendí

- **«Instalación correcta» no es «arranca»:** hay que comprobar el orden de arranque, sobre todo con dos discos.
- **Los nombres `sda`/`sdb` cambian entre arranques:** antes de escribir en un disco lo identifico por modelo (`lsblk -d -o NAME,MODEL,SIZE`) y monto por etiqueta o UUID.
- **Los pitidos y los LED del arranque son un idioma:** cada patrón es un fallo distinto (memoria, procesador, placa, BIOS…).
- **Diagnosticar por descarte:** cambiar una sola cosa cada vez (cable, disco, teclado, memoria) para saber qué era.
- **Verificar siempre los hashes** de las ISO y de lo que grabo en el pendrive.
- **Un disco de segunda mano se revisa con SMART** (sectores reasignados, pendientes e incorregibles) antes de guardar datos.
- **Un solo disco no es una copia de seguridad.**

## Siguiente

1. Sujetar el SSD y comprobar qué puerto SATA usa cada disco.
2. Docker + Portainer.
3. Copias con `rsync` + `cron` a otro destino.
4. Acceso remoto sin abrir puertos (Tailscale).
5. Pi-hole para toda la red (falta poder cambiar el DNS del router).
6. Ampliar la RAM a 16 GB (con MemTest86) y cambiar la CPU por una de 4 núcleos.

## Recursos

- HP Compaq 6005 Pro SFF, *Hardware Reference Guide*: https://h10032.www1.hp.com/ctg/Manual/c01870938.pdf
- Descarga e hashes de Ubuntu Server: https://releases.ubuntu.com/26.04/
