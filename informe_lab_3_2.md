# Laboratorio 3.2: Infraestructura de Red de una Organización con VLANs

**Universidad San Francisco Xavier de Chuquisaca**  
**Asignatura:** SIS313 – Infraestructura, Plataformas Tecnológicas y Redes  
**Docente:** Ing. Marcelo Quispe Ortega  
**Semestre:** 1/2026  

**Estudiante(s):** ______________________

---

## 1. Introducción

En este laboratorio se implementó una infraestructura virtualizada utilizando VirtualBox. Se configuró un servidor Ubuntu como router principal para realizar enrutamiento entre VLANs, acceso a internet y aplicación de políticas de seguridad mediante UFW.

---

## 2. Objetivos

- Configurar VLANs para distintos departamentos.
- Implementar enrutamiento inter-VLAN en Ubuntu Server.
- Aplicar reglas de firewall con UFW.
- Verificar conectividad entre redes.
- Restringir acceso según políticas establecidas.

---

## 3. Topología de Red

- Router-Ubuntu (VLAN 10,20,30,40)
- Server-DMZ1 (192.168.10.2)
- Server-DMZ2 (192.168.10.3)
- PC-TI (192.168.20.2)
- PC-Ventas (192.168.30.2)
- PC-Contabilidad (192.168.40.2)

Gateways configurados en el router:

- VLAN 10 → 192.168.10.1/29
- VLAN 20 → 192.168.20.1/29
- VLAN 30 → 192.168.30.1/27
- VLAN 40 → 192.168.40.1/29

> Insertar captura: Topología o máquinas virtuales creadas en VirtualBox.

---

## 4. Configuración del Router Ubuntu

Se instalaron paquetes `vlan` y `ufw`, se cargó el módulo `8021q` y se configuraron las subinterfaces `vlan10`, `vlan20`, `vlan30` y `vlan40` mediante Netplan.

```bash
sudo apt update
sudo apt install vlan ufw -y
sudo modprobe 8021q
sudo nano /etc/netplan/50-cloud-init.yaml
sudo netplan apply
```

> Insertar captura: Archivo Netplan configurado.  
> Insertar captura: Resultado de `ip addr`.

---

## 5. Habilitación de Enrutamiento

Se activó `net.ipv4.ip_forward=1` para permitir el reenvío de paquetes entre VLANs editando:

```bash
sudo nano /etc/sysctl.conf
sudo sysctl -p
```

> Insertar captura: Resultado de `sysctl -p`.

---

## 6. Configuración de Máquinas Alpine Linux

Se configuraron máquinas Alpine Linux con IP estática, gateway y etiquetado VLAN editando:

```bash
/etc/network/interfaces
```

Cada equipo recibió la IP correspondiente según su VLAN.

> Insertar captura: Archivo `/etc/network/interfaces` de una máquina cliente.

---

## 7. Verificación de Conectividad

Se verificó que la configuración del router y las VLANs estuviera correcta realizando ping a sus respectivos gateways.

Ejemplos:

```bash
ping 192.168.20.1
ping 192.168.30.1
ping 192.168.40.1
ping 192.168.10.1
```

> Insertar captura: Ping exitoso desde Ventas a su gateway.  
> Insertar captura: Ping exitoso desde Contabilidad a su gateway.

---

## 8. Configuración del Firewall UFW

Se habilitó UFW y se aplicaron reglas para permitir o denegar acceso entre VLANs según lo pedido.

```bash
sudo ufw allow ssh
sudo ufw enable
sudo ufw status numbered
```

> Insertar captura: Resultado de `sudo ufw status numbered`.

---

## 9. Pruebas Solicitadas

### Acceso a Internet

- Contabilidad tiene acceso a internet.
- Ventas no tiene acceso a internet.

> Insertar captura: Ping exitoso desde Contabilidad a 8.8.8.8.  
> Insertar captura: Ping fallido desde Ventas a 8.8.8.8.

### Acceso entre VLANs mediante SSH

Se comprobó el acceso permitido y denegado entre VLANs según políticas.  
Ejemplo: desde DMZ1 hacia TI no se logra conectividad por SSH.

> Insertar captura: SSH bloqueado desde DMZ1 hacia TI.  
> Insertar captura: SSH permitido entre VLANs autorizadas.

---

## 10. Conclusiones

- Se implementó correctamente una red segmentada mediante VLANs.
- Ubuntu Server funcionó como router y firewall.
- Se aplicaron políticas de acceso entre departamentos.
- Se verificó conectividad y restricciones según el enunciado.
- La práctica permitió comprender el uso de VLANs en redes empresariales.
