# Informe de Laboratorio 6.1  
## Automatización y Administración de Sistemas Linux con Bash Scripting

**Universidad:** Universidad Mayor, Real y Pontificia de San Francisco Xavier de Chuquisaca  
**Facultad:** Facultad de Ciencias y Tecnología  
**Carrera:** Ingeniería en Ciencias de la Computación  
**Asignatura:** SIS313 – Infraestructura, Plataformas Tecnológicas y Redes  
**Docente:** Ing. Marcelo Quispe Ortega  
**Laboratorio:** Laboratorio 6.1 – Automatización y Administración de Sistemas Linux  
**Gestión:** 1/2026  
**Grupo:** [Completar]  

### Integrantes
- Huayta Fuertes Dylan
- Gutierrez Torricos Adalid

---

# 1. Introducción

El presente laboratorio tuvo como finalidad aplicar técnicas de automatización en entornos Linux mediante Bash Scripting, utilizando Ubuntu Server como sistema operativo principal. La práctica estuvo orientada a fortalecer conocimientos relacionados con administración de sistemas, automatización de tareas repetitivas, monitoreo básico de servicios y administración remota entre servidores Linux.

Durante el desarrollo del laboratorio se trabajó con dos máquinas virtuales Ubuntu Server conectadas mediante una red Hotspot local, permitiendo implementar comunicación remota vía SSH, automatización de despliegues y verificación de servicios entre ambos nodos.

Además de la creación de scripts administrativos, se realizaron configuraciones de red estática, firewall, autenticación mediante llaves SSH y despliegue de servicios web utilizando Nginx.

---

# 2. Objetivos

## 2.1. Objetivo General

Implementar mecanismos de automatización y administración de sistemas Linux utilizando Bash Scripting en un entorno distribuido con comunicación remota segura.

## 2.2. Objetivos Específicos

- Automatizar tareas administrativas mediante scripts Bash.
- Configurar direccionamiento IP estático utilizando Netplan.
- Implementar acceso remoto seguro mediante SSH y autenticación por llaves.
- Desarrollar scripts de monitoreo y mantenimiento del sistema.
- Automatizar la creación masiva de usuarios desde archivos CSV.
- Implementar scripts de auditoría e inventario remoto.
- Realizar despliegues automatizados entre servidores Linux.

---

# 3. Topología de Red y Roles

La práctica fue desarrollada utilizando una red Hotspot local entre dos máquinas virtuales Ubuntu Server.

| Nodo | Usuario | Función | Dirección IP | Puerto SSH |
|---|---|---|---|---|
| Nodo 1 | ruls | Administrador Principal | 10.100.15.210 | 22 |
| Nodo 2 | adalidgt | Administrador Secundario | 10.100.15.211 | 2222 |

## Parámetros de Red

- **Máscara:** 255.255.255.0 (/24)
- **Gateway:** 10.100.15.179
- **DNS:** 8.8.8.8, 1.1.1.1

---

# 4. Desarrollo del Laboratorio

# FASE 1: Preparación del Entorno

---

## 4.1. Configuración de Red Estática

Para garantizar conectividad permanente entre ambos nodos, se configuraron direcciones IP estáticas mediante Netplan.

---

### Configuración realizada en Nodo 1 (ruls)

Archivo editado:

```bash
sudo nano /etc/netplan/00-installer-config.yaml
```

Configuración aplicada:

```yaml
network:
  version: 2
  ethernets:
    enp0s3:
      dhcp4: no
      addresses:
        - 10.100.15.210/24
      routes:
        - to: default
          via: 10.100.15.179
      nameservers:
        addresses:
          - 8.8.8.8
          - 1.1.1.1
```

Aplicación de cambios:

```bash
sudo netplan apply
```

---

### Configuración realizada en Nodo 2 (adalidgt)

Archivo editado:

```bash
sudo nano /etc/netplan/00-installer-config.yaml
```

Configuración aplicada:

```yaml
network:
  version: 2
  ethernets:
    enp0s3:
      dhcp4: no
      addresses:
        - 10.100.15.211/24
      routes:
        - to: default
          via: 10.100.15.179
      nameservers:
        addresses:
          - 8.8.8.8
          - 1.1.1.1
```

Aplicación de cambios:

```bash
sudo netplan apply
```

---

### Evidencia requerida

```text
[ CAPTURA 1 ]
Mostrar:
- Resultado del comando ip addr en ambos nodos.
- Comunicación exitosa mediante ping entre 10.100.15.210 y 10.100.15.211.
```

---

## 4.2. Instalación de Dependencias Base

---

### Instalación realizada en Nodo 1 (ruls)

```bash
sudo apt update
sudo apt install nginx netcat-openbsd -y
```

Pruebas locales de tráfico:

```bash
curl -s http://localhost > /dev/null
curl -s http://localhost > /dev/null
```

---

### Instalación realizada en Nodo 2 (adalidgt)

```bash
sudo apt update
sudo apt install openssh-server nginx netcat-openbsd -y
```

Habilitación del servicio SSH:

```bash
sudo systemctl start ssh
sudo systemctl enable ssh
```

---

### Cambio del puerto SSH en Nodo 2 (adalidgt)

Archivo editado:

```bash
sudo nano /etc/ssh/sshd_config
```

Parámetro modificado:

```text
Port 2222
```

Reinicio del servicio:

```bash
sudo systemctl restart ssh
```

---

### Configuración del Firewall (UFW) en Nodo 2

Para permitir el tráfico mediante el nuevo puerto configurado, se habilitó una regla específica en el firewall.

```bash
sudo ufw allow 2222/tcp
sudo ufw reload
```

---

### Evidencia requerida

```text
[ CAPTURA 2 ]
Mostrar:
- Servicio SSH funcionando en el puerto 2222.
- Resultado de sudo systemctl status ssh.
- Resultado de sudo ufw status.
```

---

## 4.3. Creación del Entorno de Trabajo (Nodo 1)

Se prepararon los directorios donde se almacenarían los scripts administrativos y respaldos.

```bash
sudo mkdir -p /opt/admin_scripts
sudo mkdir -p /var/backups/data_center
sudo chmod 755 /opt/admin_scripts
```

---

### Evidencia requerida

```text
[ CAPTURA 3 ]
Mostrar:
- Directorios creados correctamente.
- Resultado del comando ls -l /opt.
```

---

## 4.4. Intercambio de Llaves SSH

Para automatizar tareas remotas sin solicitar contraseñas, se configuró autenticación mediante llaves SSH.

---

### Generación de llaves en Nodo 1 (ruls)

```bash
ssh-keygen -t ed25519 -C "admin@lab61" -f ~/.ssh/id_lab61
```

---

### Copia de llave pública al Nodo 2

```bash
ssh-copy-id -p 2222 -i ~/.ssh/id_lab61.pub adalidgt@10.100.15.211
```

---

### Evidencia requerida

```text
[ CAPTURA 4 ]
Mostrar:
- Generación de llave SSH.
- Conexión remota exitosa sin contraseña hacia el Nodo 2.
```

---

# FASE 2: Desarrollo de Scripts Administrativos

---

## 4.5. Script de Bienvenida

Se desarrolló un script básico para registrar accesos administrativos y mostrar mensajes personalizados.

Archivo creado en Nodo 1 (ruls):

```bash
sudo nano /opt/admin_scripts/01_intro.sh
```

Código implementado:

```bash
#!/bin/bash
LOG_FILE="/tmp/admin_access.log"
NOMBRE=$1
ROL=$2

echo "========================================="
echo "¡Bienvenido, $NOMBRE! Su rol es $ROL."
echo "========================================="

echo "$(date '+%Y-%m-%d %H:%M:%S') - Usuario del sistema: $USER. Nombre: $NOMBRE, Rol: $ROL." >> $LOG_FILE

echo "Último registro añadido a $LOG_FILE:"
tail -n 1 $LOG_FILE
```

Permisos y ejecución:

```bash
sudo chmod +x /opt/admin_scripts/01_intro.sh
/opt/admin_scripts/01_intro.sh "ruls" "Administrador"
```

---

### Evidencia requerida

```text
[ CAPTURA 5 ]
Mostrar:
- Ejecución del script 01_intro.sh.
- Registro generado en /tmp/admin_access.log.
```

---

## 4.6. Verificación de Archivos y Disco

Se implementó un script para validar la existencia de archivos críticos y verificar el uso del disco.

Archivo creado:

```bash
sudo nano /opt/admin_scripts/02_check.sh
```

El script verificaba:

- Existencia del archivo de logs.
- Existencia del directorio web.
- Porcentaje de uso del disco.

---

### Evidencia requerida

```text
[ CAPTURA 6 ]
Mostrar:
- Ejecución del script 02_check.sh.
- Mensajes [OK], [ERROR] y porcentaje de disco.
```

---

## 4.7. Auditoría de Puertos TCP

Se utilizó el comando `ss` junto a pipes para identificar los puertos TCP más utilizados.

Archivo creado:

```bash
sudo nano /opt/admin_scripts/03_pipes.sh
```

---

### Evidencia requerida

```text
[ CAPTURA 7 ]
Mostrar:
- Resultado del Top 5 de puertos TCP más utilizados.
```

---

## 4.8. Análisis de Logs

Se desarrolló un script para analizar logs de Nginx utilizando estructuras `for` y `while`.

Archivo creado:

```bash
sudo nano /opt/admin_scripts/04_summarize_logs.sh
```

El script permitió:

- Contar líneas de archivos `.log`
- Contabilizar peticiones HTTP 200
- Auditar actividad web

---

### Evidencia requerida

```text
[ CAPTURA 8 ]
Mostrar:
- Conteo de logs.
- Total de peticiones HTTP 200.
```

---

## 4.9. Gestión Masiva de Usuarios vía CSV

Se automatizó la creación de grupos y usuarios leyendo datos desde un archivo CSV.

Archivo CSV:

```bash
sudo nano /opt/admin_scripts/usuarios.csv
```

Contenido:

```text
ana_sistemas,sistemas
luis_soporte,soporte
eva_sistemas,sistemas
carlos_redes,redes
```

Script implementado:

```bash
sudo nano /opt/admin_scripts/05_user_manager.sh
```

---

### Evidencia requerida

```text
[ CAPTURA 9 ]
Mostrar:
- Creación automática de grupos y usuarios.
- Mensajes [OK] y [INFO].
```

---

## 4.10. Limpieza Automatizada de Logs

Se implementó un mecanismo automático de eliminación de logs antiguos utilizando `find` y `mtime`.

Archivo creado:

```bash
sudo nano /opt/admin_scripts/log_cleanup.sh
```

---

### Evidencia requerida

```text
[ CAPTURA 10 ]
Mostrar:
- Resultado del script log_cleanup.sh.
- Reporte de limpieza generado.
```

---

## 4.11. Health Check Automático

Se desarrolló un script capaz de monitorear:

- Estado del servicio Nginx
- Uso del disco
- Memoria disponible

Archivo creado:

```bash
sudo nano /opt/admin_scripts/06_check_system.sh
```

El script reiniciaba automáticamente Nginx si detectaba fallos.

---

### Evidencia requerida

```text
[ CAPTURA 11 ]
Mostrar:
- Resultado del health check.
- Registro generado en /var/log/system_check.log.
```

---

## 4.12. Menú Interactivo de Administración

Se creó un menú interactivo para centralizar la ejecución de scripts administrativos.

Archivo creado:

```bash
sudo nano /opt/admin_scripts/07_admin_menu.sh
```

También se creó un usuario específico llamado `menu`, con privilegios sudo.

---

### Evidencia requerida

```text
[ CAPTURA 12 ]
Mostrar:
- Menú interactivo ejecutándose.
- Opciones visibles en consola.
```

---

# FASE 3: Reto Grupal

---

## 4.13. Script de Despliegue Remoto

Se creó un script para desplegar automáticamente contenido web en el servidor remoto.

Archivo creado en Nodo 1 (ruls):

```bash
nano ~/grupo_deploy.sh
```

Posteriormente se transfirió al Nodo 2 mediante SCP.

Comando utilizado:

```bash
scp -O -i ~/.ssh/id_lab61 -P 2222 ~/grupo_deploy.sh adalidgt@10.100.15.211:/tmp/grupoX_deploy.sh
```

---

### Evidencia requerida

```text
[ CAPTURA 13 ]
Mostrar:
- Transferencia exitosa del archivo mediante SCP.
- Uso del parámetro -P 2222 y -O.
```

---

## 4.14. Despliegue Remoto en Nodo 2

El administrador secundario ejecutó el script recibido.

Comandos ejecutados en Nodo 2 (adalidgt):

```bash
cd /tmp
chmod +x grupoX_deploy.sh
sudo ./grupoX_deploy.sh GrupoX Ruls Adalid
```

---

### Evidencia requerida

```text
[ CAPTURA 14 ]
Mostrar:
- Ejecución correcta del deploy.
- Sitio web generado en /var/www/.
```

---

## 4.14.b. Monitoreo Remoto de Nginx (Health Check Cruzado)

Desde el Nodo 2 se implementó un script adicional para verificar el estado del servidor web del compañero.

---

### Script creado en Nodo 2 (adalidgt)

Archivo:

```bash
sudo nano /opt/admin_scripts/health_check_cruzado.sh
```

Código implementado:

```bash
#!/bin/bash
IP_COMPANERO="10.100.15.210"

if curl -s -o /dev/null -w "%{http_code}" http://$IP_COMPANERO | grep -q "200"; then
    echo "[OK] Servidor web del compa responde correctamente."
else
    echo "[ALERTA] Servidor web del compa NO responde."
fi
```

Permisos y ejecución:

```bash
sudo chmod +x /opt/admin_scripts/health_check_cruzado.sh
/opt/admin_scripts/health_check_cruzado.sh
```

---

### Evidencia requerida

```text
[ CAPTURA EXTRA ]
Mostrar:
- Mensaje:
  [OK] Servidor web del compa responde correctamente.
```

---

## 4.15. Inventario Maestro Automatizado

Finalmente, se implementó un sistema de inventario capaz de:

- Verificar conectividad ICMP
- Validar acceso SSH
- Confirmar estado del servicio Nginx
- Generar reportes automáticos

Archivo creado en Nodo 1:

```bash
nano ~/inventory.sh
```

El script utilizó:

- `ping`
- `nc`
- `ssh`
- `systemctl`
- generación automática de reportes

---

### Evidencia requerida y análisis técnico

```text
[ CAPTURA 15 ]
Mostrar reporte generado donde se observe:

- Servidor 10.100.15.210:
  Ping OK
  SSH(22) OK

- Servidor 10.100.15.211:
  Ping OK
  SSH(22) FAIL

Nota Técnica:
El resultado [FAIL] SSH(22) en el Nodo 2 representa un falso positivo esperado, ya que el servicio SSH fue reconfigurado para trabajar exclusivamente sobre el puerto 2222 por motivos de seguridad.
```

---

# 5. Resultados Obtenidos

Durante el desarrollo del laboratorio se logró implementar exitosamente un entorno funcional de automatización y administración Linux mediante Bash Scripting.

Entre los principales resultados obtenidos destacan:

- Comunicación estable entre ambos nodos mediante red Hotspot.
- Automatización de creación de usuarios y grupos.
- Monitoreo automático del estado del sistema.
- Configuración exitosa de acceso SSH seguro.
- Transferencia remota automatizada de scripts.
- Implementación de scripts de inventario y auditoría.
- Despliegue remoto exitoso mediante SCP y SSH.

Asimismo, se fortalecieron habilidades prácticas relacionadas con troubleshooting, administración Linux y automatización de tareas reales.

---

# 6. Conclusiones

El laboratorio permitió comprender la importancia de la automatización dentro de la administración moderna de sistemas Linux. Mediante Bash Scripting fue posible reducir tareas repetitivas y centralizar procesos administrativos de manera eficiente.

La práctica también ayudó a reforzar conceptos relacionados con:

- Configuración de redes Linux
- Administración de servicios
- Seguridad básica mediante SSH
- Uso de herramientas de monitoreo
- Automatización de despliegues remotos

Finalmente, el trabajo colaborativo permitió integrar conocimientos tanto de administración local como de comunicación remota entre servidores Linux.

---

# 7. Dificultades Encontradas

Durante la práctica se presentaron algunos inconvenientes técnicos, entre ellos:

- Problemas iniciales de conectividad por configuración incorrecta de Netplan.
- Rechazo de conexiones SSH debido al cambio de puerto.
- Errores en SCP relacionados con el subsistema SFTP.
- Problemas de permisos en algunos scripts Bash.

Todos estos inconvenientes fueron solucionados mediante pruebas progresivas y revisión de logs del sistema.

---

# 8. Anexos

## Scripts Implementados

- `01_intro.sh`
- `02_check.sh`
- `03_pipes.sh`
- `04_summarize_logs.sh`
- `05_user_manager.sh`
- `log_cleanup.sh`
- `06_check_system.sh`
- `07_admin_menu.sh`
- `grupoX_deploy.sh`
- `inventory.sh`
- `health_check_cruzado.sh`

---
