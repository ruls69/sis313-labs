# INFORME DE LABORATORIO 6.1

# Automatización de Administración Linux con Bash Scripting y Despliegue Distribuido

---

# Universidad Mayor, Real y Pontificia de San Francisco Xavier de Chuquisaca

**Facultad:** Facultad de Ciencias y Tecnología
**Carrera:** Ingeniería en Ciencias de la Computación
**Asignatura:** SIS313 – Infraestructura, Plataformas Tecnológicas y Redes
**Docente:** Ing. Marcelo Quispe Ortega
**Laboratorio:** 6.1 – Automatización y Administración Linux
**Gestión:** 1/2026
**Universitario:** [TU NOMBRE]
**Grupo:** [NÚMERO DE GRUPO]

---

# 1. Introducción

El presente laboratorio tuvo como finalidad implementar mecanismos de automatización para tareas administrativas en sistemas Linux utilizando Bash Scripting sobre Ubuntu Server. La práctica se enfocó en el desarrollo de scripts para monitoreo, mantenimiento, administración de usuarios y despliegue remoto de servicios mediante SSH.

La automatización de tareas administrativas representa uno de los pilares fundamentales en la administración moderna de infraestructuras TI, ya que permite reducir errores humanos, optimizar tiempos operativos y estandarizar procedimientos críticos dentro de entornos productivos.

Durante el desarrollo de la práctica se trabajó en una arquitectura distribuida compuesta por dos máquinas físicas interconectadas mediante una red Hotspot, permitiendo realizar configuraciones de administración remota, despliegue automatizado e inventario de servicios.

---

# 2. Objetivos del Laboratorio

## 2.1 Objetivo General

Implementar soluciones automatizadas de administración y monitoreo en sistemas Linux utilizando Bash Scripting y comunicación remota segura mediante SSH.

---

## 2.2 Objetivos Específicos

* Automatizar tareas de monitoreo y mantenimiento del sistema.
* Implementar scripts de análisis de red y auditoría de servicios.
* Gestionar usuarios y grupos masivamente mediante archivos CSV.
* Configurar comunicación remota segura utilizando SSH y autenticación por llaves.
* Desplegar servicios web remotamente mediante scripts automatizados.
* Realizar inventario y monitoreo de servidores remotos.

---

# 3. Topología de Red y Entorno de Trabajo

La práctica fue desarrollada utilizando dos máquinas físicas independientes ejecutando Ubuntu Server conectadas mediante una red Hotspot local.

---

## 3.1 Arquitectura de Red

| Nodo   | Función                          | Usuario  | Dirección IP  | Puerto SSH |
| ------ | -------------------------------- | -------- | ------------- | ---------- |
| Nodo 1 | Servidor Administrador Principal | ruls     | 10.100.15.210 | 22         |
| Nodo 2 | Servidor Remoto Secundario       | adalidgt | 10.100.15.211 | 2222       |

---

## 3.2 Parámetros de Red

| Parámetro      | Valor               |
| -------------- | ------------------- |
| Gateway        | 10.100.15.179       |
| Máscara        | 255.255.255.0 (/24) |
| DNS Primario   | 8.8.8.8             |
| DNS Secundario | 1.1.1.1             |

---

📸 **CAPTURA REQUERIDA:**
Topología de red utilizada durante la práctica mostrando ambas máquinas físicas conectadas mediante Hotspot.

---

# 4. Desarrollo del Laboratorio

---

# FASE 1: Preparación del Entorno y Configuración de Red

---

## 4.1 Configuración de IP Estática – Nodo 1 (Administrador Principal)

En el Nodo 1 se configuró direccionamiento IP estático utilizando Netplan para asegurar conectividad permanente entre los nodos.

---

### Edición del archivo Netplan

*(Configuración realizada en el Nodo 1 – Usuario: ruls)*

```bash
sudo nano /etc/netplan/00-installer-config.yaml
```

### Configuración aplicada

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

### Aplicación de cambios

```bash
sudo netplan apply
```

---

📸 **CAPTURA REQUERIDA:**
Archivo Netplan configurado en el Nodo 1.

📸 **CAPTURA REQUERIDA:**
Resultado del comando `ip addr` verificando la IP 10.100.15.210.

---

## 4.2 Configuración de IP Estática – Nodo 2 (Administrador Secundario)

Se realizó la misma configuración de red en el Nodo 2 utilizando la dirección IP correspondiente.

---

### Edición del archivo Netplan

*(Configuración realizada en el Nodo 2 – Usuario: adalidgt)*

```bash
sudo nano /etc/netplan/00-installer-config.yaml
```

### Configuración aplicada

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

### Aplicación de cambios

```bash
sudo netplan apply
```

---

📸 **CAPTURA REQUERIDA:**
Archivo Netplan configurado en el Nodo 2.

📸 **CAPTURA REQUERIDA:**
Resultado del comando `ip addr` verificando la IP 10.100.15.211.

---

# 5. Instalación de Dependencias Base

---

## 5.1 Instalación de Servicios – Nodo 1

*(Instalación realizada en el Nodo 1 – Usuario: ruls)*

```bash
sudo apt update
sudo apt install nginx netcat-openbsd -y
```

### Generación de tráfico HTTP

```bash
curl -s http://localhost > /dev/null
curl -s http://localhost > /dev/null
```

---

📸 **CAPTURA REQUERIDA:**
Instalación exitosa de Nginx y Netcat en el Nodo 1.

📸 **CAPTURA REQUERIDA:**
Estado activo del servicio Nginx utilizando `systemctl status nginx`.

---

## 5.2 Instalación de Servicios – Nodo 2

*(Instalación realizada en el Nodo 2 – Usuario: adalidgt)*

```bash
sudo apt update
sudo apt install openssh-server nginx netcat-openbsd -y
```

### Habilitación del servicio SSH

```bash
sudo systemctl start ssh
sudo systemctl enable ssh
```

---

## 5.3 Configuración de SSH en Puerto Personalizado

Se modificó el servicio SSH para utilizar el puerto 2222 como medida básica de seguridad.

*(Configuración realizada en el Nodo 2 – Usuario: adalidgt)*

```bash
sudo nano /etc/ssh/sshd_config
```

### Modificación realizada

```text
Port 2222
```

### Reinicio del servicio SSH

```bash
sudo systemctl restart ssh
```

---

📸 **CAPTURA REQUERIDA:**
Archivo `sshd_config` mostrando el puerto 2222.

📸 **CAPTURA REQUERIDA:**
Resultado del comando `ss -tulnp | grep 2222`.

---

## 5.4 Configuración del Firewall (UFW) en Nodo 2

Para permitir el tráfico SSH a través del nuevo puerto configurado, se habilitó la regla correspondiente en el firewall UFW del servidor secundario.

*(Configuración realizada en el Nodo 2 – Usuario: adalidgt)*

```bash
sudo ufw allow 2222/tcp
sudo ufw reload
```

Esta configuración permitió aceptar conexiones remotas seguras únicamente por el puerto personalizado 2222/TCP, reforzando la seguridad básica del servidor al evitar el uso del puerto estándar 22.

---

📸 **CAPTURA REQUERIDA:**
Mostrar el resultado del comando `sudo ufw status` verificando la regla habilitada para el puerto 2222/TCP.

---

# 6. Creación del Entorno de Trabajo

---

## 6.1 Directorios de Administración

*(Configuración realizada en el Nodo 1 – Usuario: ruls)*

```bash
sudo mkdir -p /opt/admin_scripts
sudo mkdir -p /var/backups/data_center
sudo chmod 755 /opt/admin_scripts
```

---

📸 **CAPTURA REQUERIDA:**
Resultado del comando `ls -l /opt/`.

---

# 7. Intercambio de Llaves SSH

---

## 7.1 Generación de Llaves Criptográficas

*(Proceso realizado en el Nodo 1 – Usuario: ruls)*

```bash
ssh-keygen -t ed25519 -C "admin@lab61" -f ~/.ssh/id_lab61
```

---

## 7.2 Envío de Llave Pública al Nodo Remoto

```bash
ssh-copy-id -p 2222 -i ~/.ssh/id_lab61.pub adalidgt@10.100.15.211
```

---

## 7.3 Validación de Acceso Sin Contraseña

```bash
ssh -i ~/.ssh/id_lab61 -p 2222 adalidgt@10.100.15.211
```

---

📸 **CAPTURA REQUERIDA:**
Proceso de generación de llave SSH.

📸 **CAPTURA REQUERIDA:**
Acceso exitoso al Nodo 2 sin contraseña.

---

# 8. Desarrollo de Scripts Bash

---

## 8.1 Script de Bienvenida

*(Creado en el Nodo 1 – Usuario: ruls)*

```bash
sudo nano /opt/admin_scripts/01_intro.sh
```

### Código implementado

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

### Permisos y ejecución

```bash
sudo chmod +x /opt/admin_scripts/01_intro.sh
/opt/admin_scripts/01_intro.sh "ruls" "Administrador"
```

---

📸 **CAPTURA REQUERIDA:**
Ejecución exitosa del script `01_intro.sh`.

---

## 8.2 Script de Verificación del Sistema

*(Creado en el Nodo 1 – Usuario: ruls)*

```bash
sudo nano /opt/admin_scripts/02_check.sh
```

### Código implementado

```bash
#!/bin/bash

LOG_FILE="/tmp/admin_access.log"
DIR_WEB="/var/www/html"

if [ -f "$LOG_FILE" ]; then
    echo "[OK] El archivo de log $LOG_FILE existe."
else
    echo "[ALERTA] El archivo de log NO fue encontrado."
fi

if [ -d "$DIR_WEB" ]; then
    echo "[OK] El directorio web $DIR_WEB existe."
else
    echo "[ERROR] El directorio web $DIR_WEB no existe. Creándolo..."
    sudo mkdir -p "$DIR_WEB"
    echo "[OK] Directorio creado."
fi

USAGE=$(df -h / | awk 'NR==2 {print $5}' | sed 's/%//g')

if [ "$USAGE" -gt 85 ]; then
    echo "[CRITICO] Uso de disco: $USAGE%. Limpieza requerida."
else
    echo "[OK] Uso de disco: $USAGE%."
fi
```

### Ejecución

```bash
sudo chmod +x /opt/admin_scripts/02_check.sh
/opt/admin_scripts/02_check.sh
```

---

📸 **CAPTURA REQUERIDA:**
Ejecución completa del script `02_check.sh`.

---

## 8.3 Auditoría de Puertos TCP

*(Creado en el Nodo 1 – Usuario: ruls)*

```bash
sudo nano /opt/admin_scripts/03_pipes.sh
```

### Código implementado

```bash
#!/bin/bash

echo "Top 5 puertos TCP más utilizados:"
sudo ss -tuln | grep 'tcp ' | awk '{print $5}' | cut -d':' -f2 | sort | uniq -c | sort -nr | head -n 5
```

### Ejecución

```bash
sudo chmod +x /opt/admin_scripts/03_pipes.sh
/opt/admin_scripts/03_pipes.sh
```

---

📸 **CAPTURA REQUERIDA:**
Resultado del script `03_pipes.sh`.

---

## 8.4 Gestión Masiva de Usuarios vía CSV

*(Creado en el Nodo 1 – Usuario: ruls)*

### Creación del CSV

```bash
sudo bash -c 'echo -e "ana_sistemas,sistemas\nluis_soporte,soporte\neva_sistemas,sistemas\ncarlos_redes,redes" > /opt/admin_scripts/usuarios.csv'
```

### Script implementado

```bash
sudo nano /opt/admin_scripts/05_user_manager.sh
```

```bash
#!/bin/bash

CSV_FILE="/opt/admin_scripts/usuarios.csv"

if [ ! -f "$CSV_FILE" ]; then
    echo "[ERROR] $CSV_FILE no encontrado."
    exit 1
fi

cat "$CSV_FILE" | while IFS=',' read -r USERNAME GROUPNAME; do

    GROUPNAME=$(echo "$GROUPNAME" | tr -d '[:space:]')
    USERNAME=$(echo "$USERNAME" | tr -d '[:space:]')

    if ! grep -q "^$GROUPNAME:" /etc/group; then
        sudo groupadd "$GROUPNAME"
        echo "[OK] Grupo '$GROUPNAME' creado."
    fi

    if ! id "$USERNAME" &>/dev/null; then
        sudo useradd -m -g "$GROUPNAME" -s /bin/bash "$USERNAME"
        echo "[OK] Usuario '$USERNAME' creado."
    else
        echo "[INFO] Usuario '$USERNAME' ya existe."
    fi

done
```

### Ejecución

```bash
sudo chmod +x /opt/admin_scripts/05_user_manager.sh
sudo /opt/admin_scripts/05_user_manager.sh
```

---

📸 **CAPTURA REQUERIDA:**
Resultado exitoso del script de gestión de usuarios.

---

## 8.5 Limpieza Automatizada de Logs

*(Creado en el Nodo 1 – Usuario: ruls)*

```bash
sudo nano /opt/admin_scripts/log_cleanup.sh
```

### Código implementado

```bash
#!/bin/bash

DIAS=30
LOG_DIRS="/var/log /var/log/nginx /var/log/apache2"
REPORTE="/tmp/cleanup_report.log"

echo "[INFO] Limpieza iniciada..." > "$REPORTE"

for DIR in $LOG_DIRS; do

    if [ -d "$DIR" ]; then

        COUNT=$(find "$DIR" -type f -name "*.log*" -mtime +$DIAS 2>/dev/null | wc -l)

        if [ "$COUNT" -gt 0 ]; then
            find "$DIR" -type f -name "*.log*" -mtime +$DIAS -delete 2>/dev/null
            echo "[OK] $DIR: $COUNT logs eliminados." >> "$REPORTE"
        else
            echo "[INFO] $DIR: No hay logs mayores a $DIAS días." >> "$REPORTE"
        fi

    fi

done

cat "$REPORTE"
```

### Ejecución

```bash
sudo chmod +x /opt/admin_scripts/log_cleanup.sh
sudo /opt/admin_scripts/log_cleanup.sh
```

---

📸 **CAPTURA REQUERIDA:**
Resultado del script `log_cleanup.sh`.

---

## 8.6 Health Check del Sistema

*(Creado en el Nodo 1 – Usuario: ruls)*

```bash
sudo nano /opt/admin_scripts/06_check_system.sh
```

### Código implementado

```bash
#!/bin/bash

LOGFILE="/var/log/system_check.log"

sudo touch $LOGFILE

if ! systemctl is-active --quiet nginx; then
    echo "$(date) - ALERTA: Nginx inactivo. Reiniciando..." | sudo tee -a $LOGFILE > /dev/null
    sudo systemctl restart nginx
else
    echo "$(date) - OK: Nginx activo." | sudo tee -a $LOGFILE > /dev/null
fi

USAGE=$(df -h / | awk 'NR==2 {print $5}' | sed 's/%//g')

if [ "$USAGE" -gt 85 ]; then
    echo "$(date) - CRITICO: Disco al $USAGE%." | sudo tee -a $LOGFILE > /dev/null
else
    echo "$(date) - OK: Disco al $USAGE%." | sudo tee -a $LOGFILE > /dev/null
fi

echo "$(date) - INFO: Memoria disponible: $(free | grep Mem | awk '{print $7}')KB." | sudo tee -a $LOGFILE > /dev/null

sudo tail -n 3 $LOGFILE
```

### Ejecución

```bash
sudo chmod +x /opt/admin_scripts/06_check_system.sh
sudo /opt/admin_scripts/06_check_system.sh
```

---

📸 **CAPTURA REQUERIDA:**
Resultado del Health Check mostrando estado del disco y Nginx.

---

# 9. FASE 3 – Reto Grupal

---

## 9.1 Transferencia del Script Remoto

*(Proceso realizado desde el Nodo 1 – Usuario: ruls)*

```bash
scp -O -i ~/.ssh/id_lab61 -P 2222 ~/grupo_deploy.sh adalidgt@10.100.15.211:/tmp/grupoX_deploy.sh
```

---

📸 **CAPTURA REQUERIDA:**
Transferencia exitosa del archivo mediante `scp`.

---

## 9.2 Ejecución del Deploy Remoto

*(Proceso realizado en el Nodo 2 – Usuario: adalidgt)*

```bash
cd /tmp
chmod +x grupoX_deploy.sh
sudo ./grupoX_deploy.sh GrupoX Ruls Adalid
```

---

📸 **CAPTURA REQUERIDA:**
Resultado del despliegue remoto ejecutado exitosamente.

---

## 9.3 Monitoreo Remoto de Nginx (Health Check Cruzado)

Desde el servidor secundario (Nodo 2), se implementó un script de auditoría para verificar el estado de publicación del servidor web del Nodo 1.

---

### Script creado en Nodo 2 (adalidgt)

```bash
sudo nano /opt/admin_scripts/health_check_cruzado.sh
```

### Código implementado

```bash
#!/bin/bash

IP_COMPANERO="10.100.15.210"

if curl -s -o /dev/null -w "%{http_code}" http://$IP_COMPANERO | grep -q "200"; then
    echo "[OK] Servidor web del compa responde correctamente."
else
    echo "[ALERTA] Servidor web del compa NO responde."
fi
```

### Ejecución

```bash
sudo chmod +x /opt/admin_scripts/health_check_cruzado.sh
/opt/admin_scripts/health_check_cruzado.sh
```

---

📸 **CAPTURA EXTRA:**
Mostrar mensaje:

```text
[OK] Servidor web del compa responde correctamente.
```

---

## 9.4 Inventario Maestro Automatizado

*(Proceso realizado desde el Nodo 1 – Usuario: ruls)*

```bash
chmod +x ~/inventory.sh
./inventory.sh
```

---

## Evidencia requerida y Análisis Técnico

```text
[ CAPTURA 15 ]

Mostrar reporte generado donde se observe:

- Servidor 10.100.15.210 (Nodo 1):
  Ping OK
  SSH(22) OK

- Servidor 10.100.15.211 (Nodo 2):
  Ping OK
  SSH(22) FAIL
```

### Nota Técnica

El resultado:

```text
[FAIL] SSH(22)
```

observado en el Nodo 2 (`10.100.15.211`) representa un falso positivo esperado.

El script de inventario escanea inicialmente el puerto estándar SSH (22/TCP). Sin embargo, durante la Fase 1 del laboratorio, el servidor secundario fue reconfigurado para escuchar exclusivamente en el puerto personalizado `2222/TCP`.

Como consecuencia:

* El puerto 22 aparece inaccesible correctamente.
* El servicio SSH continúa funcionando normalmente en el puerto 2222.
* La política de seguridad aplicada fue exitosa.

---

# 10. Resultados Obtenidos

Durante el desarrollo del laboratorio se lograron implementar satisfactoriamente todos los objetivos planteados:

* Comunicación remota segura mediante SSH.
* Automatización de tareas administrativas.
* Gestión masiva de usuarios mediante CSV.
* Health checks automáticos.
* Monitoreo cruzado entre servidores.
* Transferencia remota automatizada.
* Inventario de infraestructura distribuida.

---

# 11. Conclusiones

El Laboratorio 6.1 permitió comprender la importancia de la automatización dentro de entornos Linux modernos. La implementación de scripts Bash facilitó la administración del sistema, reduciendo tiempos operativos y permitiendo realizar verificaciones automáticas sobre servicios críticos.

Asimismo, la práctica fortaleció conocimientos relacionados con:

* Administración Linux.
* Bash scripting.
* SSH y autenticación por llaves.
* Monitoreo automatizado.
* Troubleshooting de red.
* Gestión remota de infraestructura.

---

# 12. Reflexiones Finales

La automatización mediante Bash Scripting constituye una herramienta esencial para administradores de sistemas Linux. El laboratorio demostró cómo pequeños scripts pueden transformarse en soluciones completas para monitoreo, mantenimiento y despliegue distribuido.

La implementación de autenticación SSH mediante llaves y el monitoreo cruzado permitieron desarrollar una infraestructura más segura y eficiente.

---

# 13. Anexos

## 13.1 Scripts Implementados

* `01_intro.sh`
* `02_check.sh`
* `03_pipes.sh`
* `04_summarize_logs.sh`
* `05_user_manager.sh`
* `log_cleanup.sh`
* `06_check_system.sh`
* `07_admin_menu.sh`
* `grupo_deploy.sh`
* `inventory.sh`
* `health_check_cruzado.sh`

---
