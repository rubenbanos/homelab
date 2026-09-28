# Servidor 24/7 con un HP Compaq 6005 Pro reciclado

**Estado:** en curso (equipo diagnosticado y BIOS configurada; faltan los discos y el sistema)
**Fecha:** 2026-09-27
**Equipo:** HP Compaq 6005 Pro SFF (2009-2010), rescatado de un trastero

## Qué es y por qué lo hago

Quiero un servidor encendido las 24 horas, separado de mi torre (juegos) y de mi portátil (estudio), para practicar lo que se pide en soporte N1 y en el ciclo SMX: Linux sin escritorio por SSH, un NAS con Samba, Pi-hole y Docker. En vez de comprar un mini PC, aproveché dos PCs de oficina que tenía guardados y comprobé cuál servía.

## El equipo

| Dato | Valor |
|---|---|
| CPU | AMD Phenom II X2 B55, 2 núcleos a 3,0 GHz (soporta AMD-V) |
| RAM | 4 GB DDR3 (4 × 1 GB), ampliable a 16 GB |
| Placa | HP con chipset AMD 785G, BIOS 786G6 |
| Fuente | 240 W |
| Discos | ninguno de momento |
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
5. **Retirar el lector de DVD** para liberar su jaula, donde irá el SSD. Sus cables de datos y de corriente SATA se quedan para reutilizarlos.
6. **Revisar el manual del fabricante** (Hardware Reference Guide) para ver bahías, puertos SATA y tornillos, y mirar dentro de la caja antes de comprar accesorios.

## Qué se rompió

- **Problema:** el segundo PC (8000 Elite) no arrancaba: cinco pitidos y el LED rojo parpadeando cinco veces.
  - **Causa:** en la tabla de códigos de HP, cinco pitidos significan error de memoria.
  - **Solución:** de momento aparcado. Prueba pendiente: montarle un módulo del 6005 Pro; si arranca, eran sus módulos, y si no, la placa o las ranuras.
- **Problema:** pantalla negra al encender.
  - **Causa:** el monitor no estaba recibiendo señal del equipo; no era un fallo del PC.
  - **Solución:** revisar la conexión y la entrada del monitor.
- **Problema:** mi teclado mecánico de juegos no respondía en la BIOS.
  - **Causa:** las BIOS antiguas no reconocen bien los teclados gaming (protocolo distinto al de los teclados sencillos).
  - **Solución:** usar un teclado básico. Para el servidor tendré uno a mano (o uno PS/2).
- **Problema:** tras resetear la CMOS apareció el aviso 162.
  - **Causa:** el reset borra la configuración y la hora guardadas.
  - **Solución:** volver a configurar la BIOS y guardar. Falta comprobar en el próximo arranque que ya no sale.

## Resultado

El 6005 Pro arranca, la BIOS está configurada como servidor (virtualización activa, arranque automático tras un corte de luz y AHCI) y el equipo está listo para recibir discos.

Comprobación pendiente: entrar en la BIOS en el próximo arranque y confirmar que los cambios se guardaron.

## Qué aprendí

- **Los pitidos y los LED del arranque son un idioma:** cada patrón es un fallo distinto (memoria, procesador, placa, BIOS…). Los equipos de oficina llevan altavoz interno justo para eso, así que en un servidor sin pantalla lo dejo conectado.
- **Diagnosticar por descarte:** cambiar una sola cosa cada vez (cable, monitor, teclado, memoria) para saber qué era.
- **Qué hace resetear la CMOS y por qué la pila importa:** sin pila buena, la BIOS pierde la configuración y la hora.
- **AHCI, virtualización y arranque tras un corte de luz:** qué hacen y por qué las quiero en un servidor.
- **Antes de comprar accesorios, abrir la caja y mirar:** dentro ya había cables SATA, conectores de corriente libres y tornillos de repuesto.

## Siguiente

1. Montar un SSD (sistema) en la jaula del DVD y un disco duro (datos) en la bahía interna.
2. Instalar Ubuntu Server con OpenSSH, verificando antes el SHA256 de la ISO.
3. Comprobar el estado de los discos de segunda mano (`smartctl`) antes de guardar nada.
4. Samba, Pi-hole y Docker (proyectos 01-04).

## Recursos

- HP Compaq 6005 Pro SFF, *Hardware Reference Guide*: https://h10032.www1.hp.com/ctg/Manual/c01870938.pdf
