# Informe de Laboratorio 6.1: Automatización de Administración de Sistemas Linux con Bash Scripting

## Universidad Mayor, Real y Pontificia de San Francisco Xavier de Chuquisaca

### Facultad de Ciencias y Tecnología

### Carrera: Ingeniería en Ciencias de la Computación

### Asignatura: SIS313 – Infraestructura, Plataformas Tecnológicas y Redes

### Docente: Ing. Marcelo Quispe Ortega

### Laboratorio: 6.1 – Automatización de Administración Linux y Despliegue Remoto

### Gestión: 1/2026

### Modalidad: Práctica Grupal

### Integrantes:

* Ruls
* AdalidGT

---

# 1. Introducción

El presente laboratorio tuvo como finalidad aplicar técnicas de automatización sobre sistemas GNU/Linux utilizando Bash Scripting como herramienta principal de administración. Durante el desarrollo de la práctica se implementaron scripts para monitoreo, mantenimiento preventivo, auditoría de servicios, automatización de usuarios y despliegue remoto de aplicaciones.

La práctica se desarrolló utilizando dos máquinas físicas distintas conectadas mediante una red Hotspot local, ejecutando Ubuntu Server como sistema operativo base. La administración remota se realizó utilizando SSH con autenticación mediante llaves criptográficas Ed25519.

Además de la automatización local, se realizó un reto de integración grupal donde un nodo principal administró remotamente otro servidor mediante scripts automatizados, verificando conectividad, servicios activos y despliegue de contenido web.

---

# 2. Objetivos del Laboratorio

## Objetivo General

Implementar soluciones automatizadas de administración y monitoreo sobre servidores Linux utilizando Bash Scripting y herramientas nativas del sistema operativo.

## Objetivos Específicos

* Automatizar tareas administrativas mediante scripts Bash.
* Implementar verificación y monitoreo de servicios Linux.
* Gestionar usuarios y grupos utilizando archivos CSV.
* Automatizar limpieza de logs y mantenimiento preventivo.
* Configurar acceso remoto seguro mediante SSH.
* Implementar despliegue remoto automatizado entre servidores.
* Crear un sistema de inventario y auditoría de infraestructura.

---

# 3. Topología de Red y Entorno de Trabajo

Para el desarrollo del laboratorio se utilizó una red local Hotspot entre dos equipos físicos.

| Nodo   | Función                  | Usuario  | Dirección IP  | Puerto SSH |
| ------ | ------------------------ | -------- | ------------- | ---------- |
| Nodo 1 | Administrador Principal  | ruls     | 10.100.15.210 | 22         |
| Nodo 2 | Administrador Secundario | adalidgt | 10.100.15.211 | 2222       |

## Parámetros de Red

* Máscara de Subred: `255.255.255.0 (/24)`
* Gateway: `10.100.15.179`
* DNS Primario: `8.8.8.8`
* DNS Secundario: `1.1.1.1`

---

# 4. Desarrollo del Laboratorio

# FASE 1: Preparación del Entorno y Configuración de Red

---

## 4.1. Configuración de IP Estática mediante Netplan

Con el objetivo de garantizar conectividad permanente y evitar cambios dinámicos de direccionamiento IP, se configuraron direcciones IPv4 estáticas en ambos servidores utilizando Netplan.

---

### Configuración realizada en el Nodo 1 (Servidor Principal – ruls)

Se editó el archivo de configuración:

```bash
sudo nano /etc/netplan/00-installer-config.yaml
```

Contenido configurado:

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
        addresses: [8.8.8.8, 1.1.1.1]
```

Posteriormente se aplicó la configuración:

```bash
sudo netplan apply
```

---

### Configuración realizada en el Nodo 2 (Servidor Secundario – adalidgt)

Se repitió el mismo procedimiento modificando únicamente la dirección IP:

```yaml
addresses:
  - 10.100.15.211/24
```

Aplicación de cambios:

```bash
sudo netplan apply
```

---

### Evidencia requerida

```text
[ CAPTURA 1 ]
Mostrar comando:
ip addr

Debe observarse:
- IP 10.100.15.210 configurada correctamente en Nodo 1
- IP 10.100.15.211 configurada correctamente en Nodo 2
```

---

## 4.2. Instalación de Dependencias Base

Se instalaron los paquetes necesarios para el funcionamiento de los servicios web, auditoría de red y conectividad remota.

---

### Instalación realizada en el Nodo 1 (ruls)

```bash
sudo apt update
sudo apt install nginx netcat-openbsd -y
```

Se generó tráfico local para producir registros en los logs de Nginx:

```bash
curl -s http://localhost > /dev/null
curl -s http://localhost > /dev/null
```

---

### Instalación realizada en el Nodo 2 (adalidgt)

```bash
sudo apt update
sudo apt install openssh-server nginx netcat-openbsd -y
```

Se habilitó el servicio SSH:

```bash
sudo systemctl start ssh
sudo systemctl enable ssh
```

---

### Cambio del puerto SSH en Nodo 2

Se modificó el archivo:

```bash
sudo nano /etc/ssh/sshd_config
```

Se cambió:

```text
#Port 22
```

Por:

```text
Port 2222
```

Reinicio del servicio:

```bash
sudo systemctl restart ssh
```

---

### Evidencia requerida

```text
[ CAPTURA 2 ]
Mostrar:
sudo systemctl status ssh

Debe visualizarse:
- Servicio SSH activo
- Puerto configurado en 2222
```

---

# FASE 2: Desarrollo de Scripts de Administración

---

## 4.3. Creación del Entorno de Trabajo

Se creó una estructura organizada de directorios destinada al almacenamiento de scripts administrativos y respaldos del sistema.

---

### Configuración realizada en el Nodo 1 (ruls)

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
ls -ld /opt/admin_scripts
ls -ld /var/backups/data_center
```

---

## 4.4. Configuración de Acceso SSH sin Contraseña

Se implementó autenticación mediante llaves criptográficas Ed25519 para automatizar el acceso remoto seguro entre servidores.

---

### Generación de llaves (Nodo 1 – ruls)

```bash
ssh-keygen -t ed25519 -C "admin@lab61" -f ~/.ssh/id_lab61
```

---

### Copia de llave pública al Nodo 2

```bash
ssh-copy-id -p 2222 -i ~/.ssh/id_lab61.pub adalidgt@10.100.15.211
```

---

### Prueba de conexión remota

```bash
ssh -i ~/.ssh/id_lab61 -p 2222 adalidgt@10.100.15.211
```

---

### Evidencia requerida

```text
[ CAPTURA 4 ]
Mostrar:
Conexión SSH exitosa desde Nodo 1 hacia Nodo 2 sin solicitar contraseña
```

---

## 4.5. Script de Bienvenida Administrativa

Se desarrolló un script inicial encargado de registrar accesos administrativos en un archivo de logs.

---

### Script creado en Nodo 1 (ruls)

Archivo:

```bash
sudo nano /opt/admin_scripts/01_intro.sh
```

Código:

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

Permisos:

```bash
sudo chmod +x /opt/admin_scripts/01_intro.sh
```

Prueba:

```bash
/opt/admin_scripts/01_intro.sh "ruls" "Administrador"
```

---

### Evidencia requerida

```text
[ CAPTURA 5 ]
Mostrar:
Ejecución exitosa del script 01_intro.sh
y contenido generado en /tmp/admin_access.log
```

---

## 4.6. Verificación de Archivos Críticos y Disco

El script permitió verificar la existencia de archivos importantes, directorios web y el uso de disco del sistema.

---

### Script creado en Nodo 1 (ruls)

Archivo:

```bash
sudo nano /opt/admin_scripts/02_check.sh
```

---

### Funcionalidades implementadas

* Verificación del archivo `/tmp/admin_access.log`
* Validación del directorio `/var/www/html`
* Creación automática del directorio en caso de no existir
* Auditoría de uso del disco principal

---

### Ejecución

```bash
sudo chmod +x /opt/admin_scripts/02_check.sh
/opt/admin_scripts/02_check.sh
```

---

### Evidencia requerida

```text
[ CAPTURA 6 ]
Mostrar:
Resultado completo de la ejecución de 02_check.sh
```

---

## 4.7. Auditoría de Puertos TCP mediante Pipes

Se utilizó una cadena de comandos Linux para identificar los puertos TCP más utilizados del sistema.

---

### Script creado en Nodo 1 (ruls)

Archivo:

```bash
sudo nano /opt/admin_scripts/03_pipes.sh
```

---

### Comandos utilizados

* `ss`
* `grep`
* `awk`
* `cut`
* `sort`
* `uniq`

---

### Ejecución

```bash
sudo chmod +x /opt/admin_scripts/03_pipes.sh
/opt/admin_scripts/03_pipes.sh
```

---

### Evidencia requerida

```text
[ CAPTURA 7 ]
Mostrar:
Top 5 de puertos TCP detectados por el script
```

---

## 4.8. Análisis de Logs utilizando Bucles

Se desarrolló un sistema de lectura secuencial de logs mediante estructuras `for` y `while`.

---

### Script creado en Nodo 1 (ruls)

Archivo:

```bash
sudo nano /opt/admin_scripts/04_summarize_logs.sh
```

---

### Funciones implementadas

* Conteo de líneas en logs de Nginx
* Lectura automática del access.log
* Conteo de respuestas HTTP 200

---

### Ejecución

```bash
sudo chmod +x /opt/admin_scripts/04_summarize_logs.sh
sudo /opt/admin_scripts/04_summarize_logs.sh
```

---

### Evidencia requerida

```text
[ CAPTURA 8 ]
Mostrar:
Resultado del conteo de logs y peticiones HTTP 200
```

---

## 4.9. Gestión Masiva de Usuarios mediante CSV

Se automatizó la creación de usuarios y grupos utilizando un archivo CSV como base de datos estructurada.

---

### Creación del archivo CSV (Nodo 1 – ruls)

```bash
sudo bash -c 'echo -e "ana_sistemas,sistemas\nluis_soporte,soporte\neva_sistemas,sistemas\ncarlos_redes,redes" > /opt/admin_scripts/usuarios.csv'
```

---

### Script principal

Archivo:

```bash
sudo nano /opt/admin_scripts/05_user_manager.sh
```

---

### Funciones implementadas

* Lectura automatizada del CSV
* Validación de existencia de grupos
* Validación de existencia de usuarios
* Creación automática de cuentas Linux

---

### Ejecución

```bash
sudo chmod +x /opt/admin_scripts/05_user_manager.sh
sudo /opt/admin_scripts/05_user_manager.sh
```

---

### Evidencia requerida

```text
[ CAPTURA 9 ]
Mostrar:
Usuarios y grupos creados correctamente
```

---

## 4.10. Limpieza Automatizada de Logs

Se implementó un sistema de mantenimiento preventivo capaz de eliminar registros obsoletos automáticamente.

---

### Script creado en Nodo 1 (ruls)

Archivo:

```bash
sudo nano /opt/admin_scripts/log_cleanup.sh
```

---

### Funciones implementadas

* Búsqueda de logs antiguos
* Eliminación automática de archivos mayores a 30 días
* Generación de reportes de limpieza

---

### Ejecución

```bash
sudo chmod +x /opt/admin_scripts/log_cleanup.sh
sudo /opt/admin_scripts/log_cleanup.sh
```

---

### Evidencia requerida

```text
[ CAPTURA 10 ]
Mostrar:
Reporte generado por log_cleanup.sh
```

---

## 4.11. Health Check Automático

Se desarrolló un sistema de monitoreo automático del servidor y sus servicios.

---

### Script creado en Nodo 1 (ruls)

Archivo:

```bash
sudo nano /opt/admin_scripts/06_check_system.sh
```

---

### Funciones implementadas

* Verificación automática del servicio Nginx
* Reinicio automático del servicio en caso de falla
* Monitoreo de uso de disco
* Monitoreo de memoria RAM

---

### Ejecución

```bash
sudo chmod +x /opt/admin_scripts/06_check_system.sh
sudo /opt/admin_scripts/06_check_system.sh
```

---

### Evidencia requerida

```text
[ CAPTURA 11 ]
Mostrar:
Resultado del monitoreo del sistema
y contenido de /var/log/system_check.log
```

---

## 4.12. Menú Interactivo de Administración

Se creó una interfaz interactiva basada en Bash para centralizar todos los scripts administrativos.

---

### Creación del usuario administrador de menú

```bash
sudo useradd -m -s /bin/bash menu
sudo passwd menu
sudo usermod -aG sudo menu
```

---

### Script del menú

Archivo:

```bash
sudo nano /opt/admin_scripts/07_admin_menu.sh
```

---

### Funciones del menú

* Health Check
* Gestión de usuarios
* Visualización de logs
* Interfaz interactiva permanente

---

### Configuración automática al iniciar sesión

```bash
sudo bash -c 'echo "bash /opt/admin_scripts/07_admin_menu.sh" >> /home/menu/.bashrc'
```

---

### Evidencia requerida

```text
[ CAPTURA 12 ]
Mostrar:
Menú interactivo funcionando correctamente
```

---

# FASE 3: Reto Grupal – Despliegue e Inventario Remoto

---

## 4.13. Creación del Script de Despliegue Remoto

El Nodo 1 generó un script capaz de desplegar contenido HTML automáticamente en el servidor remoto.

---

### Script creado en Nodo 1 (ruls)

Archivo:

```bash
nano ~/grupo_deploy.sh
```

---

### Funciones implementadas

* Creación automática de directorios web
* Generación dinámica de archivos HTML
* Registro de despliegues realizados

---

### Transferencia al Nodo 2

```bash
scp -O -i ~/.ssh/id_lab61 -P 2222 ~/grupo_deploy.sh adalidgt@10.100.15.211:/tmp/grupoX_deploy.sh
```

---

### Evidencia requerida

```text
[ CAPTURA 13 ]
Mostrar:
Transferencia exitosa mediante SCP
```

---

## 4.14. Ejecución Remota del Despliegue

En el servidor secundario se ejecutó el script recibido desde el Nodo 1.

---

### Comandos ejecutados en Nodo 2 (adalidgt)

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
Página HTML generada correctamente
en /var/www/GrupoX
```

---

## 4.15. Inventario Maestro Automatizado

Finalmente se desarrolló un sistema centralizado de auditoría de infraestructura.

---

### Preparación de lista de servidores

```bash
echo -e "10.100.15.210\n10.100.15.211" > ~/servers.txt
```

---

### Script creado en Nodo 1 (ruls)

Archivo:

```bash
nano ~/inventory.sh
```

---

### Funciones implementadas

* Verificación ICMP (Ping)
* Verificación TCP mediante Netcat
* Verificación remota de Nginx vía SSH
* Generación de reportes automáticos

---

### Ejecución final

```bash
chmod +x ~/inventory.sh
./inventory.sh
```

---

### Evidencia requerida

```text
[ CAPTURA 15 ]
Mostrar:
Reporte final exitoso con:
- Ping OK
- SSH(2222) OK
- Nginx remoto activo
```

---

# 5. Resultados Obtenidos

Durante el desarrollo del laboratorio se logró implementar correctamente una infraestructura automatizada basada en scripts Bash. Se verificó el funcionamiento del monitoreo automático, la administración masiva de usuarios, la limpieza de logs y la auditoría remota de servicios.

La integración grupal permitió demostrar la capacidad de administración distribuida entre dos nodos Linux utilizando autenticación segura mediante llaves SSH.

---

# 6. Problemas Encontrados y Soluciones Aplicadas

| Problema                   | Solución                               |
| -------------------------- | -------------------------------------- |
| Conexión SSH rechazada     | Configuración correcta del puerto 2222 |
| Error de transferencia SCP | Uso de bandera `-O`                    |
| Permisos insuficientes     | Uso de `chmod +x` y sudo               |
| Scripts no ejecutaban      | Corrección de rutas y permisos         |
| SSH solicitaba contraseña  | Configuración de llaves Ed25519        |

---

# 7. Conclusiones

* Bash Scripting permite automatizar múltiples tareas administrativas reduciendo el tiempo operativo.
* SSH con autenticación mediante llaves mejora significativamente la seguridad y automatización.
* Linux proporciona herramientas robustas para monitoreo y auditoría de infraestructura.
* La automatización facilita el despliegue remoto y administración centralizada de servidores.
* El trabajo grupal permitió simular escenarios reales de administración de sistemas distribuidos.

---

# 8. Anexos

## Scripts Implementados

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

---

# 9. Referencias

* Documentación Oficial Ubuntu Server
* Manual de Bash Scripting GNU/Linux
* OpenSSH Documentation
* Netplan Documentation
* Nginx Official Documentation
