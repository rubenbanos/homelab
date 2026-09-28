# Redes básicas: IP, gateway y NAT en mi portátil

**Estado:** terminado
**Fecha:** 2026-09-25
**Equipo:** Portátil de estudio (Fedora 44)

## Qué es y por qué lo hago

Comprobar en mi propio portátil, con tres comandos, lo que explica el módulo 2 del curso *The Bits and Bytes of Computer Networking* (Google IT Support, Coursera): IP privada, máscara `/24`, gateway y NAT. Lo hago para entenderlo de verdad y no solo leerlo.

## Cómo lo hice

Tres comandos, en este orden. Las IPs de los ejemplos son genéricas a propósito (no subo las mías).

```bash
# 1. Mis interfaces de red y su IP privada
ip -br a
# wlo1   UP   192.168.x.x/24   ...

# 2. La tabla de rutas: quién es mi gateway
ip route
# default via 192.168.x.1 dev wlo1 proto dhcp ...

# 3. Mi IP pública, la que ve internet
curl ifconfig.me
```

Cómo leo cada resultado:

- `lo` es el loopback (mi equipo hablando consigo mismo), `eno1` es el cable (sin cable, así que `DOWN`) y `wlo1` es el wifi.
- `/24` significa que los 24 primeros bits de la IP identifican la red (máscara `255.255.255.0`): los tres primeros números son "la calle" y el último es "el piso".
- La línea `default via ...` es el gateway: a quién entrega mi portátil los paquetes cuyo destino no está en mi red. `proto dhcp` indica que esa configuración me la dio el DHCP, no la puse yo.
- La IP pública es distinta de la privada: entre las dos hay un NAT.

## Qué se rompió

Nada grave. Un detalle: `curl ifconfig.me` no termina la salida con un salto de línea, así que la IP pública aparece pegada al prompt de la terminal. Es solo aspecto; la IP es correcta.

## Resultado

Veo mi IP privada, mi gateway y mi IP pública, y sé explicar qué es cada una y por qué no coinciden. Se puede repetir en cualquier red: en otra red cambian la IP privada y la pública, y eso confirma que las asigna la red.

## Qué aprendí

- Las IPs privadas (`192.168.x.x`, `10.x.x.x`...) no son enrutables en internet, por eso se pueden repetir en todas las casas sin chocar.
- Internet nunca ve mi IP privada: el router cambia la IP al salir (NAT) y la devuelve al llegar la respuesta.
- Con `/24` solo importan los tres primeros números para saber si otro equipo está en mi misma red.
- DHCP es lo que me da IP, máscara y gateway al conectarme.

## Recursos

- Curso *The Bits and Bytes of Computer Networking* (Coursera), módulo 2.
