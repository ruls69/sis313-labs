# Proyecto Final SIS313: SOC — Centro de Operaciones de Seguridad Automatizado

> **Asignatura:** SIS313: Infraestructura, Plataformas Tecnológicas y Redes<br>
> **Semestre:** 1/2026<br>
> **Docente:** Ing. Marcelo Quispe Ortega

---

## 👥 Miembros del Equipo (Grupo 8)

| Nombre Completo | Rol en el Proyecto | Contacto (GitHub/Email) |
| :--- | :--- | :--- |
| Dylan Huayta Fuertes | Arquitecto de Infraestructura (NGINX, APP1, APP2) | [@ruls69](https://github.com/ruls69) |
| Josias Asael Quispe Ticona | Administrador de Base de Datos y Recuperación | [@J05ia5](https://github.com/J05ia5) |
| Diego Esteban Arancibia Leon | Ingeniero SOC (Monitoreo, Alertas, Automatización) | [@Diego-359](https://github.com/Diego-359) |

---

## 🎯 I. Objetivo del Proyecto

> **Objetivo:** Diseñar, implementar y demostrar una plataforma de Centro de Operaciones de Seguridad (SOC) automatizado sobre infraestructura Linux en el Centro de Datos de la Facultad, capaz de detectar ataques en tiempo real (fuerza bruta SSH, caída de servicios, eliminación de datos), generar alertas, ejecutar respuestas automáticas mediante scripts y mantener la continuidad operativa del negocio mediante backups y recuperación automatizada.

---

## 💡 II. Justificación e Importancia

> **Justificación:** Las organizaciones modernas enfrentan amenazas constantes que requieren no solo prevención, sino también detección y respuesta rápida. Este proyecto elimina la brecha entre infraestructura operativa y seguridad activa, integrando en una sola plataforma los pilares del ciclo defensivo: **Detectar → Analizar → Responder → Recuperar**. Al combinar alta disponibilidad (NGINX con failover entre APP1 y APP2), monitoreo continuo (Prometheus/Grafana), detección de intrusiones (Fail2Ban) y automatización de respuesta (scripts Bash), el proyecto resuelve el problema del *Single Point of Failure* tanto en la capa de aplicación como en la de seguridad, y demuestra cómo cada componente de infraestructura tiene un rol dentro de una arquitectura de defensa real.

---

## 🛠️ III. Tecnologías y Conceptos Implementados

### 3.1. Tecnologías Clave

* **NGINX:** Proxy inverso y balanceador de carga con Rate Limiting, HSTS y registro centralizado de accesos. Distribuye tráfico entre APP1 y APP2 con detección automática de caídas (health checks).
* **Node.js + PM2:** Servidor de aplicación web (Portal SOC de incidentes) en APP1 y APP2, gestionado por PM2 para restart automático y monitoreo de procesos.
* **MariaDB:** Base de datos central (`socdb`) que almacena usuarios, incidentes, alertas y eventos del SOC. Utilizada también para demostrar escenarios de corrupción y recuperación.
* **Prometheus + Node Exporter:** Recolección de métricas del sistema (CPU, RAM, red, disponibilidad de servicios) desde todas las VMs.
* **Grafana:** Dashboard principal del SOC. Visualiza métricas en tiempo real, genera alertas visuales ante incidentes (caída de nodo, picos de tráfico, servicios offline).
* **Fail2Ban:** Detección y bloqueo automático de ataques de fuerza bruta SSH. Monitorea `/var/log/auth.log`.
* **Scripts Bash SOC:** Automatización de health checks, respuesta a incidentes, backup/restore de base de datos y generación de reportes.
* **Hydra / Nmap / stress-ng (VM6):** Herramientas de simulación de ataques para los escenarios demostrativos de la feria.

### 3.2. Conceptos de la Asignatura Puestos en Práctica (T1 – T6)

* ✅ **Alta Disponibilidad (T2) y Tolerancia a Fallos:** NGINX balancea carga entre APP1 (192.168.208.3) y APP2 (192.168.208.4). Ante la caída de cualquiera de ellas, el tráfico se redirige automáticamente a la disponible. Escenario 4 (apagar APP2) lo demuestra en vivo con alerta en Grafana.
* ✅ **Seguridad y Hardening (T5):** Fail2Ban con reglas personalizadas en `jail.local`, SSL/TLS en NGINX, acceso SSH restringido, Rate Limiting HTTP.
* ✅ **Automatización y Gestión (T6):** Scripts de backup (`mysqldump` + compresión + rotación), restore automático de base de datos, health check multi-servicio, respuesta automática a incidentes y menú interactivo SOC Command Center.
* ✅ **Balanceo de Carga / Proxy Inverso (T3/T4):** NGINX como punto de entrada único distribuyendo tráfico con upstream hacia APP1 y APP2, incluyendo registro de accesos y Rate Limiting por IP.
* ✅ **Monitoreo en Tiempo Real (T4/T1):** Prometheus recolecta métricas vía Node Exporter en cada VM. Grafana presenta dashboards con estado de servicios, tráfico, alertas críticas y estado del clúster.
* ✅ **Networking y Segmentación (T3):** VLAN configurada en la red 192.168.208.0/24 como capa de red interna del clúster, con IPs físicas en 192.168.100.0/24. Cada VM tiene rol definido dentro de la topología.

---

## 🌐 IV. Diseño de la Infraestructura y Topología

### 4.1. Diagrama de Topología

```
                           INTERNET
                               │
                               │
                  ┌──────────────────────────────┐
                  │  NGINX Reverse Proxy         │
                  │  Load Balancer               │
                  │  IP Física: 192.168.100.168  │
                  │  IP VLAN:  192.168.208.2     │
                  │  Usuario:  adming8           │
                  └──────────┬───────────────────┘
                             │
              ┌──────────────┼──────────────┐
              │                             │
   ┌──────────▼──────────┐     ┌───────────▼─────────┐
   │       APP1          │     │        APP2          │
   │  IP: 100.169/208.3  │     │  IP: 100.170/208.4   │
   │  Node.js + PM2      │     │  Node.js + PM2       │
   └──────────┬──────────┘     └───────────┬──────────┘
              │                             │
              └──────────────┬──────────────┘
                             │
                ┌────────────▼─────────────┐
                │         MariaDB          │
                │  IP: 100.171 / 208.5     │
                │  Base de Datos: socdb    │
                │  Tablas: usuarios,       │
                │  incidentes, alertas     │
                └────────────┬─────────────┘
                             │  Logs / Métricas
                             ▼
             ┌───────────────────────────────────┐
             │          SOC SERVER               │
             │  IP: 100.172 / 208.6              │
             │  • Grafana (Dashboard)            │
             │  • Prometheus + Node Exporter     │
             │  • Fail2Ban                       │
             │  • Scripts de respuesta           │
             │  • SOC Command Center (menú)      │
             └───────────────────────────────────┘
                             │
             ┌───────────────────────────────────┐
             │     BACKUP & ATTACK SERVER        │
             │  IP: 100.173 / 208.7              │
             │  Modo Atacante: Hydra, Nmap       │
             │  Modo Recuperación: mysqldump,    │
             │  restore scripts, backups         │
             └───────────────────────────────────┘
```

### 4.2. Tabla de Infraestructura

| VM / Host | Rol | IP Física | IP VLAN | Usuario | SO |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **VM1 – NGINX** | Reverse Proxy + Load Balancer | 192.168.100.168 | 192.168.208.2 | adming8 | Ubuntu 24.04.4 LTS |
| **VM2 – APP1** | Servidor de Aplicación 1 (Node.js + PM2) | 192.168.100.169 | 192.168.208.3 | adming8 | Ubuntu 24.04.4 LTS |
| **VM3 – APP2** | Servidor de Aplicación 2 (Node.js + PM2) | 192.168.100.170 | 192.168.208.4 | adming8 | Ubuntu 24.04.4 LTS |
| **VM4 – DB** | Base de Datos MariaDB Central | 192.168.100.171 | 192.168.208.5 | adming8 | Ubuntu 24.04.4 LTS |
| **VM5 – SOC Server** | Monitoreo, Detección y Respuesta | 192.168.100.172 | 192.168.208.6 | adming8 | Ubuntu 24.04.4 LTS |
| **VM6 – Backups** | Backups, Restore y Simulación de Ataques | 192.168.100.173 | 192.168.208.7 | adming8 | Ubuntu 24.04.4 LTS |

### 4.3. Estrategia de Diseño

* **Estrategia de Alta Disponibilidad:** NGINX actúa como único punto de entrada y distribuye tráfico entre APP1 y APP2 mediante upstream con health checks pasivos. Si una aplicación deja de responder, NGINX redirige el 100% del tráfico a la disponible sin intervención manual.
* **Estrategia de Seguridad en Capas:** La defensa está distribuida en tres niveles: perímetro (NGINX con Rate Limiting), autenticación (Fail2Ban en SSH), e integridad de datos (backups automáticos con verificación de integridad y restore script).
* **Estrategia SOC Narrativa:** En lugar de presentar servidores estáticos, el proyecto se estructura como una narrativa de incidente real: el equipo asume el rol de analistas SOC respondiendo a ataques en vivo, lo que hace la demostración comprensible y atractiva para el público de la feria.

---

## 📋 V. Guía de Implementación y Puesta en Marcha

### 5.1. Pre-requisitos

* 6 VMs con Ubuntu 24.04.4 LTS, acceso root/sudo, y conectividad en la red VLAN 192.168.208.0/24.
* Repositorio del proyecto clonado en cada VM.
* NGINX, Node.js, PM2, MariaDB, Prometheus, Grafana, Fail2Ban instalados en sus respectivas VMs.
* Hydra, Nmap y stress-ng instalados en VM6 (Backup & Attack Server).

### 5.2. Configuración por VM

La configuración utilizada en los servidores siguió la siguiente estructura:

```yaml
network:
  version: 2
  renderer: networkd

  ethernets:
    ens18:
      addresses:
        - 192.168.100.169/24

      routes:
        - to: default
          via: 192.168.100.1

  vlans:
    vlan208:
      id: 208
      link: ens18

      addresses:
        - 192.168.208.X/28
```

La utilización de una VLAN dedicada permitió aislar la infraestructura del proyecto respecto a otros grupos alojados dentro de la misma supercomputadora.

---
# Configuración de Hostnames y Resolución de Nombres

Con el objetivo de simplificar la administración y facilitar futuras modificaciones de infraestructura, se configuraron nombres lógicos para cada servidor.

Ejemplos:

```bash
hostnamectl set-hostname nginx-lb
hostnamectl set-hostname app1
hostnamectl set-hostname app2
```

Posteriormente se configuró resolución local mediante el archivo:

```bash
/etc/hosts
```

Agregando las siguientes entradas:

```text
192.168.208.2 nginx-lb
192.168.208.3 app1
192.168.208.4 app2
192.168.208.5 mariadb
192.168.208.6 monitoreo
192.168.208.7 backup
```

Gracias a esta configuración fue posible utilizar nombres de host dentro de NGINX y los scripts administrativos, evitando el uso constante de direcciones IP.

---
# Implementación del Balanceador de Carga NGINX

La máquina virtual nginx-lb fue configurada como punto único de entrada para todas las solicitudes realizadas por los usuarios.

La instalación se realizó mediante:

```bash
sudo apt update
sudo apt install nginx -y
```

La configuración principal fue almacenada en:

```bash
/etc/nginx/sites-available/soc
```

y posteriormente habilitada mediante:

```bash
ln -s /etc/nginx/sites-available/soc /etc/nginx/sites-enabled/soc
```

La validación de la sintaxis se realizó utilizando:

```bash
nginx -t
```

y la configuración fue aplicada mediante:

```bash
systemctl reload nginx
```

---
## Configuración del Upstream

Para implementar balanceo de carga se creó un grupo de servidores denominado:

```nginx
upstream soc_backend {

    least_conn;

    server app1:3000 max_fails=3 fail_timeout=30s;
    server app2:3000 max_fails=3 fail_timeout=30s;

}
```
#### Failover Automático

Cada backend fue configurado con:

```nginx
max_fails=3
fail_timeout=30s
```

Si un servidor presenta tres errores consecutivos durante treinta segundos, NGINX deja de enviarle tráfico temporalmente.

Esto permite mantener la disponibilidad incluso cuando una aplicación presenta fallos.

---
# Implementación de HTTPS

Con el objetivo de proteger las comunicaciones entre clientes y servidores se configuró HTTPS.

Los certificados utilizados fueron almacenados en:

```text
/etc/ssl/certs/soc.crt
/etc/ssl/private/soc.key
```

La redirección automática fue configurada mediante:

```nginx
server {

    listen 80;

    return 301 https://$host$request_uri;

}
```

Posteriormente se habilitó HTTPS mediante:

```nginx
listen 443 ssl http2;
```

y se restringieron los protocolos permitidos:

```nginx
ssl_protocols TLSv1.2 TLSv1.3;
```

Con esta configuración se eliminaron protocolos inseguros y se garantizó que todo el tráfico viaje cifrado.

---

# Hardening del Servidor Web

Como parte del fortalecimiento de la superficie de exposición se aplicaron diversas configuraciones de hardening.

Se ocultó la versión de NGINX mediante:

```nginx
server_tokens off;
```

para evitar que un atacante identifique fácilmente la versión utilizada.

También se implementaron cabeceras de seguridad:

```nginx
add_header Strict-Transport-Security "max-age=31536000" always;
add_header X-Frame-Options "DENY" always;
add_header X-Content-Type-Options "nosniff" always;
add_header Referrer-Policy "strict-origin" always;
add_header X-XSS-Protection "1; mode=block" always;
add_header Permissions-Policy "geolocation=()" always;
```

Estas políticas ayudan a mitigar ataques de Clickjacking, XSS y filtración de información.

---

# Protección contra Reconocimiento y Escaneo

Como parte del enfoque Detectar y Responder se implementó una política básica de bloqueo de herramientas ofensivas.

La configuración aplicada fue:

```nginx
if ($http_user_agent ~* "(sqlmap|nikto|nmap|masscan|wpscan)") {
    return 403;
}
```

Cuando una solicitud contiene alguno de estos User-Agent, NGINX responde automáticamente:

```text
403 Forbidden
```

Las pruebas fueron realizadas mediante:

```bash
curl -A "sqlmap" https://192.168.208.2 -k
```

obteniendo el resultado esperado.

---

# Implementación de Rate Limiting

Con el objetivo de reducir el impacto de ataques automatizados se implementó limitación de solicitudes.

Configuración:

```nginx
limit_req_zone $binary_remote_addr zone=soclimit:10m rate=5r/s;
```

y posteriormente:

```nginx
limit_req zone=soclimit burst=10 nodelay;
```

La política permite:

* 5 solicitudes por segundo por IP.
* Ráfagas controladas de hasta 10 solicitudes.
* Reducción del impacto de ataques básicos de denegación de servicio.

---

# Preparación para Monitoreo

Con el objetivo de integrar Grafana y Prometheus se habilitó el módulo Stub Status.

Configuración:

```nginx
location /nginx_status {

    stub_status;

    allow 192.168.208.6;
    deny all;

}
```

Esta funcionalidad permite exponer métricas relacionadas con:

* Conexiones activas.
* Solicitudes procesadas.
* Estado operativo de NGINX.

El acceso fue restringido exclusivamente al servidor de monitoreo.

---

# Desarrollo del Portal SOC Incident Portal

## Estructura del Proyecto

La aplicación fue desplegada en APP1 y APP2 dentro del directorio:

```bash
/opt/soc-app
```

Inicialmente el portal mostraba únicamente información estática.

Posteriormente se decidió transformarlo en una herramienta de validación de infraestructura capaz de verificar simultáneamente:

* Estado de APP1 y APP2.
* Funcionamiento del balanceador.
* Disponibilidad de MariaDB.
* Visualización de datos reales almacenados en la base de datos.

---

## Instalación de Dependencias

Se instaló NodeJS:

```bash
sudo apt install nodejs npm -y
```

Posteriormente se instaló la librería de conexión a MariaDB:

```bash
cd /opt/soc-app

npm install mysql2
```

---

## Información Mostrada por el Portal

La aplicación fue diseñada para mostrar información útil durante las pruebas operativas.

Elementos mostrados:

* Backend activo.
* Hostname del servidor.
* Estado de la aplicación.
* Estado de la base de datos.
* Total de usuarios registrados.
* Fecha y hora del servidor.
* Tabla completa de usuarios.

Esta información permite validar visualmente el correcto funcionamiento de toda la infraestructura.

---

## Identificación del Backend Activo

Para verificar el funcionamiento del balanceador se implementó identificación dinámica del servidor que responde cada solicitud.

La aplicación utiliza:

```javascript
curl hhtp://192.168.100.168
```

permitiendo visualizar:

```text
Backend APP1
```

o

```text
Backend APP2
```

según el servidor seleccionado por NGINX.

---

## Integración con MariaDB

La aplicación establece conexión con:

```text
Servidor: 192.168.208.5
Base de Datos: socdb
Tabla: usuarios
```

Cada vez que un usuario accede al portal se ejecuta:

```sql
SELECT * FROM usuarios;
```

mostrando información real almacenada en la base de datos.

Campos visualizados:

* id
* nombre
* correo
* edad
* fecha_registro

---

# Administración de Aplicaciones con PM2

Con el objetivo de mantener disponibilidad continua se utilizó PM2.

Instalación:

```bash
npm install -g pm2
```

Despliegue:

```bash
pm2 start app.js --name app1

pm2 save

pm2 startup
```

Administración:

```bash
pm2 list
pm2 restart app1
pm2 stop app1
pm2 logs app1
```

PM2 permite reinicio automático ante fallos y persistencia tras reinicios del sistema.

---

# Automatización Operativa mediante Bash

Todos los scripts fueron almacenados en:

```bash
/opt/soc
```

El objetivo fue reducir tareas manuales y facilitar la operación del SOC.

---

## app_status.sh

Función:

* Consultar APP1.
* Consultar APP2.
* Verificar estado PM2 remotamente.

Código:

```bash
#!/bin/bash

echo "====== APP STATUS ======"

ssh app1 "pm2 list"

echo

ssh app2 "pm2 list"
```

---

## health_check.sh

Función:

* Verificar APP1.
* Verificar APP2.
* Verificar NGINX.

Código:

```bash
#!/bin/bash

echo "=== HEALTH CHECK ==="

curl -s http://app1:3000 > /dev/null

if [ $? -eq 0 ]
then
    echo "APP1 OK"
else
    echo "APP1 DOWN"
fi

curl -s http://app2:3000 > /dev/null

if [ $? -eq 0 ]
then
    echo "APP2 OK"
else
    echo "APP2 DOWN"
fi

systemctl is-active nginx
```

---

## lb_status.sh

Función:

* Verificar NGINX.
* Mostrar conexiones activas.

Código:

```bash
#!/bin/bash

echo "====== LOAD BALANCER ======"

systemctl status nginx --no-pager

echo
echo "Conexiones activas"

ss -ant | grep ':80' | wc -l
```

---

# Desarrollo del SOC Command Center

Con el objetivo de centralizar todas las tareas operativas se desarrolló una consola administrativa propia denominada:

```text
SOC COMMAND CENTER
```

Ubicación:

```bash
/opt/soc/soc_menu.sh
```

Antes de su implementación era necesario ejecutar manualmente múltiples comandos para verificar el estado de la infraestructura.

La consola fue desarrollada para actuar como una capa de orquestación sobre los scripts previamente creados.

Al ejecutarse:

```bash
bash /opt/soc/soc_menu.sh
```

presenta un menú interactivo con opciones de monitoreo y administración.

Funciones integradas:

1. Estado de Aplicaciones.
2. Health Check.
3. Estado del Balanceador.
4. Visualización de Logs.
5. Consulta de Conexiones Activas.
6. Salida del sistema.

La herramienta permite realizar verificaciones rápidas sin necesidad de recordar comandos individuales.

Durante la feria tecnológica será utilizada como consola principal de operación y demostración del SOC.

---

# Automatización mediante Llaves SSH

Inicialmente los scripts requerían ingreso manual de contraseñas.

Para automatizar completamente la ejecución se implementó autenticación mediante llaves SSH.

Generación:

```bash
ssh-keygen -t ed25519
```

Distribución:

```bash
ssh-copy-id ruls@app1
ssh-copy-id ruls@app2
```

Validación:

```bash
ssh app1
ssh app2
```

Gracias a esta configuración fue posible ejecutar consultas remotas desde nginx-lb sin intervención del operador.

---

# Pruebas Realizadas

Las pruebas efectuadas sobre la infraestructura implementada fueron:

| Prueba               | Resultado |
| -------------------- | --------- |
| Balanceo APP1 ↔ APP2 | Exitosa   |
| Failover APP1        | Exitosa   |
| HTTPS                | Exitosa   |
| TLS 1.2/1.3          | Exitosa   |
| Integración MariaDB  | Exitosa   |
| Consulta de usuarios | Exitosa   |
| PM2                  | Exitosa   |
| SSH Keys             | Exitosa   |
| Rate Limiting        | Exitosa   |
| Bloqueo SQLMap       | Exitosa   |
| nginx_status         | Exitosa   |
| SOC Command Center   | Exitosa   |

Los resultados obtenidos demostraron el correcto funcionamiento de la infraestructura implementada y su integración con el resto de componentes del SOC.

**VM4 – Base de Datos (MariaDB)**

```sql
-- Creación de base de datos SOC
CREATE DATABASE socdb;
USE socdb;
CREATE TABLE `usuarios` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `nombre` varchar(100) NOT NULL,
  `correo` varchar(150) DEFAULT NULL,
  `edad` int(11) DEFAULT NULL,
  `fecha_registro` timestamp NULL DEFAULT current_timestamp(),
  PRIMARY KEY (`id`),
  UNIQUE KEY `correo` (`correo`)
)
```

**VM5 – SOC Server**

```bash
# Configuración de Fail2Ban
# /etc/fail2ban/jail.local
[sshd]
enabled  = true
maxretry = 5
bantime  = 3600
findtime = 600

# Verificar estado de Fail2Ban
sudo fail2ban-client status sshd

# Desbloquear IP manualmente (para resetear demo)
sudo fail2ban-client set sshd unbanip 192.168.208.7
```

**VM6 – Backup & Attack Server**

```bash
# Script de backup automático
# /backups/backup_database.sh
#!/bin/bash

DATE=$(date +%F_%H-%M)

mysqldump \
-h 192.168.208.5 \
-u backup \
-pjosias \
socdb > /backups/database/$DATE.sql

gzip /backups/database/$DATE.sql


# Script de restore
# /backups/recuperar.sh
#!/bin/bash

# Configuración de variables (T1 / T15)
DIR_RESPALDOS="/backups/database"
DB_HOST="192.168.208.5"
DB_USER="backup"
DB_PASS="josias"
DB_NAME="socdb"

echo "===================================================="
echo "      SISTEMA DE RECUPERACIÓN ANTE DESASTRES - SOC  "
echo "===================================================="

# 1. Listar los respaldos disponibles para que el usuario elija
echo "=== Respaldos disponibles en el sistema:"
# Opción corregida y robusta:
ls -1 "$DIR_RESPALDOS"/*.sql.gz 2>/dev/null | xargs -L 1 basename
echo "----------------------------------------------------"

# 2. Solicitar al administrador qué archivo usar
read -p "- Escribe el nombre exacto del archivo a restaurar (ej: 2026-06-07_17-14.sql.gz): " ARCHIVO_ELEGIDO

RUTA_COMPLETA="$DIR_RESPALDOS/$ARCHIVO_ELEGIDO"

# Validar que el archivo realmente exista
if [ ! -f "$RUTA_COMPLETA" ]; then
    echo "!!! x Error: El archivo '$ARCHIVO_ELEGIDO' no existe."
    exit 1
fi

echo "... Iniciando restauración de la base de datos desde la VLAN..."

# 3. La magia de la recuperación en una sola línea sin extraer en disco de forma permanente (T14)
# 'zcat' lee el contenido comprimido al vuelo y lo envía por tubería '|' al cliente de mysql remoto
zcat "$RUTA_COMPLETA" | mysql -h "$DB_HOST" -u "$DB_USER" -p"$DB_PASS" "$DB_NAME"

# 4. Verificación del estado de salida del comando anterior
if [ ${PIPESTATUS[1]} -eq 0 ]; then
    echo "===================================================="
    echo "✓ ¡ÉXITO! La base de datos '$DB_NAME' ha sido restaurada."
    echo "   Estado: Operacional a partir del respaldo: $ARCHIVO_ELEGIDO"
    echo "===================================================="
else
    echo "!!! Error crítico: Falló la inyección del respaldo en el servidor remoto."
fi
```

### 5.3. Ficheros de Configuración Clave

| Ruta | Descripción |
| :--- | :--- |
| `/etc/nginx/sites-available/socshield.conf` | Configuración del proxy inverso y balanceo de carga |
| `/etc/fail2ban/jail.local` | Reglas de detección y bloqueo de fuerza bruta SSH |
| `/etc/prometheus/prometheus.yml` | Scrape targets de todas las VMs |
| `/backups/backup_database.sh` | Backup automático de MariaDB con compresión |
| `/backups/recuperar.sh` | Restauración automática de la base de datos |
| `/opt/soc/health_check.sh` | Verificación de estado de todos los servicios |
| `/opt/soc/soc_menu.sh` | Menú interactivo SOC Command Center |

### 5.4. SOC Command Center (Menú Interactivo)

```text
==================================
      SOC COMMAND CENTER
==================================
 [1]  Estado General
 [2]  Incidentes Activos
 [3]  Ver Alertas
 [4]  Ejecutar Backup
 [5]  Restaurar Backup
 [6]  Health Check General
 [7]  Estado de Aplicaciones
 [8]  Estado de Base de Datos
 [9]  Estado de Fail2Ban
 [10] Estado de Firewall
 [11] Ver Logs Centralizados
 [12] Generar Reporte
 [13] Respuesta Automática
 [14] Simular Ataque
 [15] Salir
==================================
```

---

## ⚠️ VI. Pruebas y Validación — Escenarios de Ataque Demostrados

### Escenario 1: Ataque de Fuerza Bruta SSH

**Objetivo:** Demostrar detección y bloqueo automático de intentos de login masivos.

**Ejecución desde VM6 (Atacante):**

```bash
hydra -l adming8 -P passwords.txt ssh://192.168.208.2
```

**Monitoreo en NGINX (VM1):**

```bash
sudo tail -f /var/log/auth.log
sudo fail2ban-client status sshd
```

**Reset de demo:**

```bash
sudo fail2ban-client set sshd unbanip 192.168.208.7
```

| Prueba Realizada | Resultado Esperado | Resultado Obtenido |
| :--- | :--- | :--- |
| Fuerza bruta SSH desde VM6 con Hydra | Fail2Ban bloquea la IP atacante tras 5 intentos fallidos; alerta en Grafana | ✅ OK |

---

### Escenario 2: Caída de Aplicación (APP1)

**Objetivo:** Demostrar failover automático del balanceador de carga.

**Ejecución en APP1 (VM2):**

```bash
pm2 list          # Verificar estado previo
pm2 stop app1     # Simular caída del servicio
```

**Verificación:**
El sitio web continúa funcionando a través de APP2. Grafana muestra APP1 como OFFLINE y el failover activado.

**Restauración:**

```bash
pm2 start app1    # Restaurar servicio
```

| Prueba Realizada | Resultado Esperado | Resultado Obtenido |
| :--- | :--- | :--- |
| Detener APP1 con `pm2 stop app1` | NGINX redirige todo el tráfico a APP2; sitio sigue operativo; Grafana marca APP1 OFFLINE | ✅ OK |

---

### Escenario 3: Corrupción de Base de Datos

**Objetivo:** Demostrar detección de incidente de integridad y recuperación automática.

**Ejecución en DB (VM4) — primero mostrar logs, luego ejecutar:**

```bash
# Mostrar estado previo y logs
sudo tail -f /var/log/mysql/error.log
```

```sql
-- Simular corrupción de datos
sudo mysql
USE socdb;
DROP TABLE usuarios;
SELECT * FROM usuarios;  -- Error: tabla no existe
```

**Detención del servicio:**

```bash
sudo systemctl stop mariadb
```

**Restauración desde VM6:**

```bash
bash /opt/soc/restore_database.sh
```

| Prueba Realizada | Resultado Esperado | Resultado Obtenido |
| :--- | :--- | :--- |
| `DROP TABLE usuarios` + `systemctl stop mariadb` | SOC detecta pérdida de datos; script restore_database.sh recupera la tabla desde backup | ✅ OK |

---

### Escenario 4: Apagado Completo de APP2

**Objetivo:** Demostrar monitoreo Grafana y continuidad operativa ante caída total de nodo.

**Ejecución en APP2 (VM3):**

```bash
sudo shutdown -h now
```

**Resultado:** Grafana detecta que la VM dejó de enviar métricas (Node Exporter offline) y genera alerta visual de nodo caído. El servicio continúa operativo a través de APP1.

| Prueba Realizada | Resultado Esperado | Resultado Obtenido |
| :--- | :--- | :--- |
| Apagado total de VM3 (APP2) | Grafana muestra nodo offline; tráfico concentrado en APP1; sistema continúa operativo | ✅ OK |

---

### Resumen de Pruebas

| # | Escenario | Herramienta / Comando | Mecanismo de Defensa | Resultado |
| :--- | :--- | :--- | :--- | :--- |
| 1 | Fuerza bruta SSH | `hydra -l adming8 -P passwords.txt ssh://192.168.208.2` | Fail2Ban → banip automático | ✅ OK |
| 2 | Caída de APP1 | `pm2 stop app1` | NGINX failover a APP2 | ✅ OK |
| 3 | Corrupción de BD | `DROP TABLE usuarios` + `systemctl stop mariadb` | Script de restore automático | ✅ OK |
| 4 | Apagado de APP2 | `sudo shutdown -h now` | Grafana alerta + failover a APP1 | ✅ OK |

---

## 📚 VII. Conclusiones y Lecciones Aprendidas

El proyecto **SOC** logró integrar exitosamente los conceptos fundamentales de la asignatura SIS313 en una arquitectura coherente y funcional orientada a la ciberseguridad defensiva. Los principales logros técnicos fueron:

* La implementación de un ciclo completo **Detectar → Analizar → Responder → Recuperar** sobre infraestructura Linux real, con cada VM cumpliendo un rol específico dentro de la cadena defensiva.
* La demostración en vivo de cuatro escenarios de incidente, mostrando que el sistema es capaz de responder automáticamente sin intervención humana en la mayoría de los casos.
* La integración de herramientas profesionales de monitoreo (Prometheus/Grafana) con herramientas de seguridad (Fail2Ban) y automatización (scripts Bash), logrando un SOC funcional a escala universitaria.

**Desafíos superados:**

* La configuración de los health checks de NGINX para detectar caídas de APP1/APP2 de forma inmediata sin afectar el rendimiento en condiciones normales.
* La coordinación del equipo para que los escenarios de ataque y respuesta se ejecutaran fluidamente durante la feria.

**¿Qué haríamos diferente?**

* Implementar el escenario de integridad de archivos con hashes SHA256 (similar a Tripwire) para añadir un quinto vector de detección.
* Añadir alertas por correo electrónico o webhook (Slack/Telegram) desde Grafana para simular notificación real al equipo SOC.
* Explorar la replicación Maestro-Esclavo en MariaDB para llevar la alta disponibilidad también a la capa de base de datos.

> *"No presentamos servidores. Presentamos una narrativa: somos el equipo de respuesta a incidentes de una empresa. Un atacante intentó comprometer nuestros sistemas en tiempo real. Nuestro SOC detectó la amenaza, generó alertas, ejecutó respuestas automáticas y mantuvo la continuidad operativa del negocio."*

---

*Informe — SIS313: Infraestructura, Plataformas Tecnológicas y Redes — Universidad San Francisco Xavier de Chuquisaca — Semestre 1/2026*
