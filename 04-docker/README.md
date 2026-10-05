# Docker y servicios

**Estado:** en curso (Docker, Portainer cerrado al exterior y Uptime Kuma funcionando; faltan más monitores y avisos)
**Fecha:** 2026-10-04 y 2026-10-05
**Equipo:** servidor HP Compaq 6005 Pro con Ubuntu Server 26.04 (ver [`06-servidor-hp-6005-pro`](../06-servidor-hp-6005-pro/))

## Qué es y por qué lo hago

Docker mete cada programa en una caja aislada (un contenedor) con todo lo que necesita dentro, así que puedo arrancar un servicio con un comando y borrarlo sin dejar rastro ni pisar a los demás. Aparece en casi todas las ofertas de sistemas y DevOps, y es la base para montar el resto de servicios del servidor. Portainer es un panel web para ver y manejar los contenedores sin memorizar comandos. Uptime Kuma vigila que mis servicios (router, Pi-hole, NAS) estén vivos y me lo enseña en un panel.

## Cómo lo hice

1. **Instalar Docker** desde los repositorios de Ubuntu (`docker.io`). Descarté `snap` porque aísla Docker del sistema y da problemas con volúmenes y permisos.

```bash
sudo apt install docker.io
```

2. **Probar que funciona** con la imagen de prueba oficial, que imprime un mensaje y termina:

```bash
sudo docker run hello-world
```

3. **Crear un volumen** para que los datos de Portainer sobrevivan si borro el contenedor:

```bash
sudo docker volume create portainer_data
```

4. **Arrancar Portainer:**

```bash
sudo docker run -d --name portainer --restart always \
  -p 9443:9443 \
  -v /var/run/docker.sock:/var/run/docker.sock \
  -v portainer_data:/data \
  portainer/portainer-ce:lts
```

| Parte | Qué hace |
|---|---|
| `-d` | Lo deja en segundo plano |
| `--name portainer` | Le pone nombre al contenedor |
| `--restart always` | Arranca solo tras un reinicio o un corte de luz |
| `-p 9443:9443` | Conecta el puerto 9443 del servidor con el 9443 del contenedor (la web) |
| `-v /var/run/docker.sock:…` | Le da el «mando» de Docker para manejar los demás contenedores |
| `-v portainer_data:/data` | Guarda sus datos en el volumen |
| `portainer/portainer-ce:lts` | Imagen gratuita (`ce`) y estable (`lts`) |

5. **Comprobar** que está en marcha con `sudo docker ps` y entrar desde el navegador del portátil a `https://<ip-del-servidor>:9443` (el navegador avisa del certificado porque Portainer usa uno propio).
6. **Crear el administrador** con el token de un solo uso que Portainer escribe en sus registros (`sudo docker logs portainer`) y una contraseña larga guardada en el gestor de contraseñas.
7. **Limpiar:** borrar desde Portainer el contenedor de `hello-world`, que ya no hace falta.

### 5 de octubre: cerrar Portainer al exterior

Al publicar con `-p 9443:9443`, Portainer escuchaba en todas las interfaces y Docker se saltaba ufw (ver «Qué aprendí»). Lo recreé para que solo escuche dentro del propio servidor:

```bash
sudo docker stop portainer
sudo docker rm portainer
sudo docker run -d --name portainer --restart always \
  -p 127.0.0.1:9443:9443 \
  -v /var/run/docker.sock:/var/run/docker.sock \
  -v portainer_data:/data \
  portainer/portainer-ce:lts
```

Borrar el contenedor no borra el volumen `portainer_data`, así que conservo el usuario administrador. Ahora el panel solo se alcanza con un **túnel SSH** desde mi portátil:

```bash
ssh -L 9443:localhost:9443 ruben@<ip-del-servidor>
```

Con el túnel abierto entro en `https://localhost:9443`. `-L 9443:localhost:9443` abre el puerto 9443 de mi portátil y lo lleva por SSH hasta el 9443 del servidor. Para comprobar que quedó cerrado, desde otro equipo de la red hice `nmap -Pn -p 9443 <ip-del-servidor>`: pasó de abierto a `filtered`.

### 5 de octubre: Uptime Kuma

Un segundo contenedor, publicado **solo en la IP del servidor** y no en todas las interfaces:

```bash
sudo docker run -d --restart=unless-stopped \
  -p <ip-del-servidor>:3001:3001 \
  -v uptime-kuma:/app/data \
  --name uptime-kuma louislam/uptime-kuma:2
```

En el asistente web elegí **SQLite** como base de datos (ligera, adecuada para un servidor con poca RAM) y creé el administrador. Después añadí tres monitores, comprobando cada 60 segundos:

| Monitor | Tipo | Qué vigila |
|---|---|---|
| Router | Ping | Que el router responde |
| Pi-hole | HTTP | Que el panel de Pi-hole carga |
| NAS Samba | Puerto TCP | Que el puerto 445 del servidor acepta conexiones |

## Qué se rompió

- **Problema:** `pull access denied for portainer/portainer-ce.lts, repository does not exist`.
  - **Causa:** escribí un punto en vez de dos puntos en la última línea. En Docker, `:` separa el nombre de la imagen de su etiqueta, y con el punto buscaba una imagen distinta que no existe.
  - **Solución:** corregir a `portainer-ce:lts`. Como el error ocurrió al descargar, no se creó ningún contenedor a medias.
- **Problema:** la web de Portainer mostraba «Your Portainer instance timed out for security purposes».
  - **Causa:** Portainer se bloquea si pasan unos 5 minutos sin crear el administrador, y yo tardé más.
  - **Solución:** `sudo docker restart portainer`, tener la contraseña lista de antemano y crear el usuario enseguida. Pasó dos veces.
- **Problema:** error 404 al abrir la web.
  - **Causa:** la instancia seguía sin administrador. Lo comprobé con la API (`/api/users/admin/check` devuelve 404 mientras no exista ninguno).
  - **Solución:** reiniciar, sacar el token nuevo de los registros y repetir el alta.
- **Problema:** escribí la dirección `https://…:9443` en la terminal y dio `No such file or directory`.
  - **Causa:** una URL va en la barra del navegador, no en la terminal.
  - **Solución:** abrirla en Chrome.

- **Problema:** Uptime Kuma marcaba Pi-hole como caído: `timeout of 48000ms exceeded`.
  - **Causa 1:** puse `https://` y el servidor solo atiende por `http` (el 443 está cerrado).
  - **Causa 2 (la importante):** con `http://` seguía caído. Kuma vive dentro de un contenedor, y su tráfico hacia el propio servidor no llega desde mi red de casa sino desde la **red interna de Docker** (interfaz `docker0`). ufw solo permitía mi red local, así que descartaba esos paquetes **en silencio**, y de ahí el timeout en vez de un «connection refused».
  - **Solución:** una regla de ufw por cada puerto que Kuma vigila, con origen la red de Docker:

```bash
sudo ufw allow from <red-interna-de-docker> to any port 80 proto tcp
sudo ufw allow from <red-interna-de-docker> to any port 445 proto tcp
```

    El origen se ve con `ip -br a` (interfaz `docker0`). Tras las reglas, Pi-hole y el NAS pasaron a «Funcional».

## Resultado

Docker 29.1 funcionando. Portainer arranca solo con el servidor, **ya no se ve desde la red** y entro con un túnel SSH. Uptime Kuma vigila router, Pi-hole y NAS y me da el porcentaje de disponibilidad de cada uno (sube con el tiempo: es la fracción de comprobaciones correctas desde que lo creé). Lo compruebo con `sudo docker ps` y entrando a la web. Un contenedor que sigue vivo (`running`) es un servicio; uno que termina su trabajo queda en `exited - code 0`, como `hello-world`.

## Qué aprendí

- **Imagen, contenedor y volumen:** la imagen es la plantilla de solo lectura, el contenedor es esa imagen en marcha y el volumen es la carpeta que sobrevive si borro el contenedor. Si borro un contenedor, pierdo lo que había dentro salvo lo que esté en un volumen.
- **Un contenedor vive mientras viva su programa principal.** `hello-world` imprime y termina; Portainer es un servicio que no termina.
- **Docker se salta ufw.** Al publicar un puerto con `-p`, Docker escribe sus propias reglas de red antes que las de ufw. Portainer responde en el 9443 aunque `ufw status` no lo permita, así que no puedo fiarme de ufw para los puertos de Docker. Aquí solo es visible en mi red de casa, y nunca debo abrir ese puerto en el router. La solución limpia (pendiente) es publicarlo solo en local con `-p 127.0.0.1:9443:9443` y entrar por un túnel SSH.
- **Un contenedor que llama al propio servidor llega desde la red de Docker, no desde mi LAN.** Una regla de firewall es puerto + origen + acción; si falta la del origen correcto, ufw descarta sin avisar y el síntoma es un **timeout** (un rechazo explícito sería «connection refused»).
- **Publicar con `127.0.0.1:`** delante del puerto (`-p 127.0.0.1:9443:9443`) hace que el servicio solo escuche en local. Para verlo desde fuera, túnel SSH; no hace falta abrir nada.
- **Con cada contenedor nuevo, mirar `docker ps` y la columna `PORTS`**, no `ufw status`: ahí está lo que de verdad queda expuesto.
- **Acceso a `docker.sock` equivale a control total del servidor,** por eso Portainer solo debe verse desde mi red.
- **Fijarme en el prompt** (`ruben@fedora` o `ruben@homelab`) antes de ejecutar nada: un comando en la máquina equivocada da resultados que despistan.

## Recursos

- [Instalar Portainer CE con Docker en Linux](https://docs.portainer.io/start/install-ce/server/docker/linux)
- [Documentación de Docker](https://docs.docker.com/)
- [Docker y ufw](https://docs.docker.com/engine/network/packet-filtering-firewalls/)

## Pendiente

- Más monitores en Kuma (Portainer, SSH) y avisos por correo o Telegram.
