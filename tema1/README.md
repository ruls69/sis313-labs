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
                  ┌────────────────────────┐
                  │  NGINX Reverse Proxy   │
                  │  Load Balancer         │
                  │  IP Física: 192.168.100.168  │
                  │  IP VLAN:  192.168.208.2     │
                  │  Usuario:  adming8      │
                  └──────────┬─────────────┘
                             │
              ┌──────────────┼──────────────┐
              │                             │
   ┌──────────▼──────────┐     ┌───────────▼─────────┐
   │       APP1          │     │        APP2          │
   │  IP: 100.169/208.3  │     │  IP: 100.170/208.4  │
   │  Node.js + PM2      │     │  Node.js + PM2       │
   └──────────┬──────────┘     └───────────┬──────────┘
              │                             │
              └──────────────┬──────────────┘
                             │
                ┌────────────▼─────────────┐
                │         MariaDB          │
                │  IP: 100.171 / 208.5     │
                │  Base de Datos: socdb    │
                │  Tablas: usuarios,        │
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
| **VM1 – NGINX** | Reverse Proxy + Load Balancer | 192.168.100.168 | 192.168.208.2 | adming8 | Ubuntu 22.04 |
| **VM2 – APP1** | Servidor de Aplicación 1 (Node.js + PM2) | 192.168.100.169 | 192.168.208.3 | adming8 | Ubuntu 22.04 |
| **VM3 – APP2** | Servidor de Aplicación 2 (Node.js + PM2) | 192.168.100.170 | 192.168.208.4 | adming8 | Ubuntu 22.04 |
| **VM4 – DB** | Base de Datos MariaDB Central | 192.168.100.171 | 192.168.208.5 | adming8 | Ubuntu 22.04 |
| **VM5 – SOC Server** | Monitoreo, Detección y Respuesta | 192.168.100.172 | 192.168.208.6 | adming8 | Ubuntu 22.04 |
| **VM6 – Backups** | Backups, Restore y Simulación de Ataques | 192.168.100.173 | 192.168.208.7 | adming8 | Ubuntu 22.04 |

### 4.3. Estrategia de Diseño

* **Estrategia de Alta Disponibilidad:** NGINX actúa como único punto de entrada y distribuye tráfico entre APP1 y APP2 mediante upstream con health checks pasivos. Si una aplicación deja de responder, NGINX redirige el 100% del tráfico a la disponible sin intervención manual.
* **Estrategia de Seguridad en Capas:** La defensa está distribuida en tres niveles: perímetro (NGINX con Rate Limiting), autenticación (Fail2Ban en SSH), e integridad de datos (backups automáticos con verificación de integridad y restore script).
* **Estrategia SOC Narrativa:** En lugar de presentar servidores estáticos, el proyecto se estructura como una narrativa de incidente real: el equipo asume el rol de analistas SOC respondiendo a ataques en vivo, lo que hace la demostración comprensible y atractiva para el público de la feria.

---

## 📋 V. Guía de Implementación y Puesta en Marcha

### 5.1. Pre-requisitos

* 6 VMs con Ubuntu 22.04 LTS, acceso root/sudo, y conectividad en la red VLAN 192.168.208.0/24.
* Repositorio del proyecto clonado en cada VM.
* NGINX, Node.js, PM2, MariaDB, Prometheus, Grafana, Fail2Ban instalados en sus respectivas VMs.
* Hydra, Nmap y stress-ng instalados en VM6 (Backup & Attack Server).


### 5.2. Configuración por VM

**VM1 – NGINX (Proxy + Balanceador)**

```nginx
# /etc/nginx/sites-available/socshield.conf
upstream socapp {
    server 192.168.208.3:3000;
    server 192.168.208.4:3000;
}

server {
    listen 80;
    location / {
        proxy_pass http://socapp;
        limit_req zone=one burst=20 nodelay;
    }
}
```

**VM2 y VM3 – APP1 y APP2 (Node.js + PM2)**

```bash
# Iniciar aplicación con PM2
pm2 start app.js --name app1
pm2 save
pm2 startup

# Verificar estado
pm2 list
```

**VM4 – Base de Datos (MariaDB)**

```sql
-- Creación de base de datos SOC
CREATE DATABASE socdb;
USE socdb;
CREATE TABLE usuarios (id INT AUTO_INCREMENT PRIMARY KEY, nombre VARCHAR(100), rol VARCHAR(50));
CREATE TABLE incidentes (id INT AUTO_INCREMENT PRIMARY KEY, tipo VARCHAR(100), fecha DATETIME, estado VARCHAR(50));
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
# /opt/soc/backup_manager.sh
mysqldump -h 192.168.208.5 -u root -p socdb > /backups/socdb_$(date +%Y%m%d_%H%M).sql
gzip /backups/socdb_$(date +%Y%m%d_%H%M).sql

# Script de restore
# /opt/soc/restore_database.sh
gunzip -c /backups/socdb_latest.sql.gz | mysql -h 192.168.208.5 -u root -p socdb
```

# 5.2.1 Implementación de Infraestructura de Aplicaciones, Balanceo de Carga y Seguridad Perimetral (Integrante 1)

El Integrante 1 fue responsable del diseño e implementación de toda la capa de acceso al sistema, incluyendo la red de aplicaciones, el balanceador de carga NGINX, los servidores Node.js, la integración con MariaDB, los mecanismos de seguridad perimetral y la automatización operativa mediante scripts Bash.

La finalidad de esta capa fue proporcionar disponibilidad continua, distribución eficiente de carga, comunicaciones seguras y mecanismos básicos de detección y respuesta frente a incidentes.

---

# Diseño de la Red de Aplicaciones

El proyecto fue desplegado dentro de la VLAN 208 asignada al grupo de trabajo.

Cada máquina virtual fue configurada utilizando Netplan y una interfaz VLAN dedicada.

La distribución final fue la siguiente:

| Máquina Virtual | Dirección IP VLAN 208 | Función                                          |
| --------------- | --------------------- | ------------------------------------------------ |
| nginx-lb        | 192.168.208.2         | Balanceador de carga y punto de acceso principal |
| app1            | 192.168.208.3         | Servidor de aplicación Node.js                   |
| app2            | 192.168.208.4         | Servidor de aplicación Node.js                   |
| mariadb         | 192.168.208.5         | Base de datos centralizada                       |
| monitoreo       | 192.168.208.6         | Grafana y Prometheus                             |
| backup          | 192.168.208.7         | Servidor de copias de seguridad                  |

La configuración de red aplicada siguió la siguiente estructura:

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
        - 192.168.208.2/28
```

La utilización de una VLAN dedicada permitió aislar los servicios del proyecto respecto a otros grupos alojados dentro de la misma infraestructura física.

---

# Configuración de Hostnames y Resolución de Nombres

Con el objetivo de facilitar la administración y simplificar las configuraciones se asignaron nombres lógicos a cada servidor.

Ejemplos:

```bash
hostnamectl set-hostname nginx-lb

hostnamectl set-hostname app1

hostnamectl set-hostname app2
```

Posteriormente se configuró resolución local mediante:

```bash
/etc/hosts
```

agregando:

```text
192.168.208.2 nginx-lb
192.168.208.3 app1
192.168.208.4 app2
192.168.208.5 mariadb
192.168.208.6 monitoreo
```

Gracias a esta configuración el balanceador pudo utilizar nombres de host en lugar de direcciones IP.

---

# Implementación del Balanceador de Carga

La máquina virtual nginx-lb fue configurada como punto único de entrada para todos los usuarios del sistema.

Se instaló NGINX:

```bash
sudo apt update

sudo apt install nginx -y
```

Posteriormente se creó la configuración principal:

```bash
sudo nano /etc/nginx/sites-available/soc
```

La definición del grupo de servidores backend fue la siguiente:

```nginx
upstream soc_backend {

    least_conn;

    server app1:3000 max_fails=3 fail_timeout=30s;
    server app2:3000 max_fails=3 fail_timeout=30s;

}
```

El algoritmo Least Connections fue seleccionado debido a que distribuye las nuevas solicitudes al servidor con menor cantidad de conexiones activas.

Esta estrategia proporciona un uso más equilibrado de los recursos cuando existen diferencias temporales de carga entre servidores.

---

# Implementación del Reverse Proxy

NGINX fue configurado para actuar como intermediario entre los clientes y las aplicaciones.

```nginx
location / {

    limit_req zone=soclimit burst=10 nodelay;

    proxy_pass http://soc_backend;

    proxy_http_version 1.1;

    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header X-Forwarded-Proto $scheme;

}
```

Esta configuración permite:

* Balanceo transparente.
* Conservación de la IP real del cliente.
* Registro correcto de eventos.
* Compatibilidad con aplicaciones web modernas.

---

# Implementación de HTTPS

Con el objetivo de proteger la información intercambiada entre usuarios y servidores se configuró HTTPS.

La redirección automática fue implementada mediante:

```nginx
server {

    listen 80;

    return 301 https://$host$request_uri;

}
```

Las conexiones seguras fueron configuradas mediante:

```nginx
listen 443 ssl http2;
```

y:

```nginx
ssl_protocols TLSv1.2 TLSv1.3;
```

Se eliminaron protocolos inseguros y se restringió el uso a versiones modernas de TLS.

---

# Hardening del Servidor NGINX

Como parte del fortalecimiento de seguridad se aplicaron medidas de hardening.

Se ocultó la versión del servidor:

```nginx
server_tokens off;
```

Se configuraron cabeceras de protección:

```nginx
add_header Strict-Transport-Security "max-age=31536000" always;

add_header X-Frame-Options "DENY" always;

add_header X-Content-Type-Options "nosniff" always;

add_header Referrer-Policy "strict-origin" always;

add_header X-XSS-Protection "1; mode=block" always;

add_header Permissions-Policy "geolocation=()" always;
```

Estas configuraciones reducen la exposición frente a ataques basados en navegador.

---

# Implementación de Protección contra Escaneo

Se configuró un mecanismo básico de identificación de herramientas ofensivas mediante User-Agent.

```nginx
if ($http_user_agent ~* "(sqlmap|nikto|nmap|masscan|wpscan)") {
    return 403;
}
```

La finalidad de esta medida es bloquear automáticamente herramientas frecuentemente utilizadas durante fases de reconocimiento.

Durante las pruebas se verificó la respuesta utilizando:

```bash
curl -A "sqlmap" https://192.168.208.2 -k
```

obteniendo como resultado:

```text
403 Forbidden
```

---

# Implementación de Rate Limiting

Para mitigar ataques de fuerza bruta o abuso de solicitudes se implementó control de tasa.

```nginx
limit_req_zone $binary_remote_addr zone=soclimit:10m rate=5r/s;
```

y posteriormente:

```nginx
limit_req zone=soclimit burst=10 nodelay;
```

La política final permite:

* 5 solicitudes por segundo por dirección IP.
* Ráfagas controladas de hasta 10 solicitudes.
* Reducción del impacto de ataques automatizados.

---

# Preparación para Monitoreo

Con el objetivo de integrar Prometheus y Grafana se habilitó Stub Status.

```nginx
location /nginx_status {

    stub_status;

    allow 192.168.208.6;

    deny all;

}
```

Las métricas expuestas incluyen:

* Conexiones activas.
* Solicitudes procesadas.
* Estado general del servicio web.

El acceso fue restringido exclusivamente al servidor de monitoreo.

---

# Implementación de APP1 y APP2

Se desplegaron dos servidores de aplicación independientes utilizando Node.js.

La instalación se realizó mediante:

```bash
sudo apt install nodejs npm -y
```

Las aplicaciones fueron configuradas para escuchar en:

```text
Puerto 3000
```

Cada instancia ejecuta el portal denominado:

```text
SOC INCIDENT PORTAL
```

---

# Integración con MariaDB

La aplicación fue desarrollada para conectarse dinámicamente a la base de datos centralizada.

Se instaló la librería:

```bash
npm install mysql2
```

La aplicación establece conexión con:

```text
Base de Datos: socdb
Tabla: usuarios
```

Mostrando:

* ID
* Nombre
* Correo electrónico
* Edad
* Fecha de registro

La página también informa:

* Backend activo.
* Hostname.
* Estado de aplicación.
* Estado de base de datos.
* Cantidad total de usuarios.

Esta funcionalidad permite validar simultáneamente el funcionamiento del balanceador, la aplicación y la base de datos.

---

# Administración mediante PM2

Las aplicaciones fueron gestionadas utilizando PM2.

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

PM2 proporciona reinicio automático ante fallos y persistencia tras reinicios del sistema.

---

# Automatización Operativa mediante Scripts Bash

Con el objetivo de reducir tareas manuales se desarrolló un conjunto de herramientas administrativas ubicadas en:

```text
/opt/soc
```

## app_status.sh

Consulta remotamente el estado de las aplicaciones.

```bash
#!/bin/bash

echo "====== APP STATUS ======"

ssh app1 "pm2 list"

echo

ssh app2 "pm2 list"
```

---

## health_check.sh

Verifica disponibilidad de servicios.

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

Permite consultar el estado operativo del balanceador.

```bash
#!/bin/bash

echo "====== LOAD BALANCER ======"

systemctl status nginx --no-pager

echo
echo "Conexiones activas"

ss -ant | grep ':80' | wc -l
```

---

## soc_menu.sh

Se desarrolló una consola administrativa denominada:

```text
SOC COMMAND CENTER
```

La herramienta centraliza todas las tareas operativas implementadas durante el proyecto.

Permite:

* Estado de aplicaciones.
* Health Check.
* Estado del balanceador.
* Consulta de logs.
* Verificación de conexiones activas.

---

# Automatización mediante Llaves SSH

Inicialmente los scripts requerían autenticación manual mediante contraseña.

Para automatizar completamente el proceso se implementó autenticación basada en llaves SSH.

Generación:

```bash
ssh-keygen -t ed25519
```

Distribución:

```bash
ssh-copy-id usuario@app1

ssh-copy-id usuario@app2
```

Esta configuración permitió ejecutar consultas remotas sin intervención manual, facilitando el funcionamiento de los scripts desarrollados.

---

# Pruebas Realizadas

Las pruebas efectuadas sobre la infraestructura implementada fueron:

* Balanceo APP1 ↔ APP2.
* Acceso HTTPS.
* Verificación TLS.
* Conectividad con MariaDB.
* Consulta dinámica de usuarios.
* Reinicio de aplicaciones mediante PM2.
* Failover ante caída de APP1.
* Bloqueo de User-Agent malicioso.
* Verificación de scripts Bash.
* Consulta de métricas mediante nginx_status.

Los resultados obtenidos demostraron el correcto funcionamiento de la infraestructura desarrollada y su integración con el resto de componentes del SOC.

### 5.3. Ficheros de Configuración Clave

| Ruta | Descripción |
| :--- | :--- |
| `/etc/nginx/sites-available/socshield.conf` | Configuración del proxy inverso y balanceo de carga |
| `/etc/fail2ban/jail.local` | Reglas de detección y bloqueo de fuerza bruta SSH |
| `/etc/prometheus/prometheus.yml` | Scrape targets de todas las VMs |
| `/opt/soc/backup_manager.sh` | Backup automático de MariaDB con compresión |
| `/opt/soc/restore_database.sh` | Restauración automática de la base de datos |
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

*Informe generado para la Feria de Proyectos — SIS313: Infraestructura, Plataformas Tecnológicas y Redes — Universidad San Francisco Xavier de Chuquisaca — Semestre 1/2026*
