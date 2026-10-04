# Pi-hole: bloqueo de anuncios y DNS local

**Estado:** en curso (instalado y probado; falta que toda la red lo use)
**Fecha:** 2026-10-03
**Equipo:** servidor HP Compaq 6005 Pro con Ubuntu Server 26.04

## Qué es y por qué lo hago

Pi-hole es un servidor DNS que bloquea anuncios y rastreadores para toda la red de casa. Lo monto para entender cómo funciona el DNS por dentro: qué pasa cuando un equipo pide un dominio y quién le contesta.

## Cómo lo hice

1. Instalar Pi-hole v6 en el servidor (con IP fija, imprescindible para un servidor DNS).
2. Elegir un DNS de salida (Cloudflare) y una lista de bloqueo (StevenBlack, unos 72.000 dominios).
3. Abrir el puerto 53 (DNS) y el 80 (panel web) en `ufw`, solo para mi red local.
4. **Probar** desde mi torre apuntando su DNS al servidor:

```bash
nslookup doubleclick.net
# Address: 0.0.0.0   <- bloqueado
```

El panel mostró las consultas llegando y bloqueadas.

## Qué se rompió

- **Problema:** el DNS de Pi-hole no se usaba en toda la red.
  - **Causa:** para que lo usen todos los aparatos hay que cambiar el DNS que reparte el router, y no tengo la contraseña de administrador del router (el operador de la fibra no me la dio).
  - **Solución:** pendiente. Opciones: pedirla al operador, o poner un router propio detrás del suyo. No reseteo el router: perdería la configuración de la fibra.
- **Problema:** aun apuntando Windows a Pi-hole, parte de las consultas se saltaban el bloqueo.
  - **Causa:** el router también reparte un DNS por IPv6 y Windows prefiere ese.
  - **Solución:** cubrir IPv4 **e** IPv6 (en la prueba, desactivé IPv6 en el adaptador). Lo dejé revertido hasta poder tocar el router.

## Resultado

Pi-hole funciona: resuelve, bloquea y registra consultas. Ahora mismo no tiene clientes fijos.

## Qué aprendí

- **Pi-hole solo cuenta lo que le llega:** si el router sigue repartiendo otro DNS, no ve nada.
- **IPv4 e IPv6 son dos caminos distintos** para el DNS.
- **Si el servidor se apaga y es el único DNS, los aparatos se quedan sin internet:** hace falta un segundo DNS de reserva.
- **Un servidor DNS necesita IP fija.**

## Recursos

- Documentación de Pi-hole: https://docs.pi-hole.net/
