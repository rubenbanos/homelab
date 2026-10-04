# Docker y servicios

**Estado:** en curso (Docker y Portainer funcionando; falta Uptime Kuma)
**Fecha:** 2026-10-04
**Equipo:** servidor HP Compaq 6005 Pro con Ubuntu Server 26.04 (ver [`06-servidor-hp-6005-pro`](../06-servidor-hp-6005-pro/))

## Qué es y por qué lo hago

Docker mete cada programa en una caja aislada (un contenedor) con todo lo que necesita dentro, así que puedo arrancar un servicio con un comando y borrarlo sin dejar rastro ni pisar a los demás. Aparece en casi todas las ofertas de sistemas y DevOps, y es la base para montar el resto de servicios del servidor. Portainer es un panel web para ver y manejar los contenedores sin memorizar comandos.

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

## Resultado

Docker 29.1 funcionando, Portainer arranca solo con el servidor y desde el panel veo contenedores, imágenes, volúmenes y redes. Lo compruebo con `sudo docker ps` y entrando a la web. Un contenedor que sigue vivo (`running`) es un servicio; uno que termina su trabajo queda en `exited - code 0`, como `hello-world`.

## Qué aprendí

- **Imagen, contenedor y volumen:** la imagen es la plantilla de solo lectura, el contenedor es esa imagen en marcha y el volumen es la carpeta que sobrevive si borro el contenedor. Si borro un contenedor, pierdo lo que había dentro salvo lo que esté en un volumen.
- **Un contenedor vive mientras viva su programa principal.** `hello-world` imprime y termina; Portainer es un servicio que no termina.
- **Docker se salta ufw.** Al publicar un puerto con `-p`, Docker escribe sus propias reglas de red antes que las de ufw. Portainer responde en el 9443 aunque `ufw status` no lo permita, así que no puedo fiarme de ufw para los puertos de Docker. Aquí solo es visible en mi red de casa, y nunca debo abrir ese puerto en el router. La solución limpia (pendiente) es publicarlo solo en local con `-p 127.0.0.1:9443:9443` y entrar por un túnel SSH.
- **Acceso a `docker.sock` equivale a control total del servidor,** por eso Portainer solo debe verse desde mi red.
- **Fijarme en el prompt** (`ruben@fedora` o `ruben@homelab`) antes de ejecutar nada: un comando en la máquina equivocada da resultados que despistan.

## Recursos

- [Instalar Portainer CE con Docker en Linux](https://docs.portainer.io/start/install-ce/server/docker/linux)
- [Documentación de Docker](https://docs.docker.com/)
- [Docker y ufw](https://docs.docker.com/engine/network/packet-filtering-firewalls/)

## Pendiente

- Uptime Kuma para vigilar que los servicios estén activos.
- Publicar Portainer solo en local y entrar por túnel SSH.
