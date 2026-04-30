# Laboratorio 3.2: Infraestructura de Red de una Organización con VLANs

Universidad San Francisco Xavier de Chuquisaca  
Asignatura: SIS313 – Infraestructura, Plataformas Tecnológicas y Redes  
Docente: Ing. Marcelo Quispe Ortega  
Semestre: 1/2026

---

## Introducción

En este laboratorio se implementó una infraestructura virtualizada utilizando VirtualBox. Se configuró un servidor Ubuntu como router principal para realizar enrutamiento entre VLANs, acceso a internet y aplicación de políticas de seguridad mediante UFW.

---

## Topología de Red

- Router-Ubuntu (VLAN 10,20,30,40)
- Server-DMZ1 (192.168.10.2)
- Server-DMZ2 (192.168.10.3)
- PC-TI (192.168.20.2)
- PC-Ventas (192.168.30.2)
- PC-Contabilidad (192.168.40.2)

Con los gateways configurados en el router de la siguiente manera.

> Insertar captura

---

## Configuración

Se instalaron paquetes `vlan` y `ufw`, se cargó el módulo `8021q` y se configuraron las subinterfaces `vlan10`, `vlan20`, `vlan30` y `vlan40` mediante Netplan.

> Insertar captura

Se activó `net.ipv4.ip_forward=1` para permitir el reenvío de paquetes entre VLANs entrando a `/etc/sysctl.conf`.

> Insertar captura

Se configuraron máquinas Alpine Linux con IP estática, gateway y etiquetado VLAN entrando en `/etc/network/interfaces`.

> Insertar captura

---

## Verificación

Verificamos que la configuración del router y VLANs esté correcta con ping a sus respectivos gateways.

> Insertar captura

Se habilitó UFW y se aplicaron reglas para permitir o denegar acceso entre VLANs según lo pedido.

> Insertar captura

---

## Pruebas

Contabilidad tiene acceso a internet.

> Insertar captura

Ventas no tiene acceso a internet.

> Insertar captura

SSH permitido/denegado entre VLANs según políticas. Se comprueba haciendo SSH desde DMZ1 a TI y no se logra la conectividad.

> Insertar captura

---

## Conclusiones

Se implementó correctamente una red segmentada mediante VLANs.  
Ubuntu Server funcionó como router y firewall.  
Se aplicaron políticas de acceso entre departamentos.
