# INFORME DE LABORATORIO: SERVICIOS DE RED (DNS Y WEB)

**Materia:** SIS313 - Infraestructura de Redes y Servicios  
**Estudiante:** Huayta Fuertes Dylan  
**Docente:** Ing. Marcelo Quispe Ortega  

---

## 1. Configuración del servidor DNS

Primero se configuró la parte del servidor DNS, el cual funcionará como servidor con salida a internet.  
Para ello, se habilitaron **dos adaptadores de red**:

- **Adaptador NAT:** destinado a la salida a internet.
- **Adaptador Interno:** utilizado para la comunicación interna entre las demás máquinas virtuales.
- <img width="886" height="295" alt="image" src="https://github.com/user-attachments/assets/19fb76dd-4940-4ca6-8275-1e8637dafc28" />
- <img width="863" height="409" alt="image" src="https://github.com/user-attachments/assets/342a791d-0134-4cc8-828c-04bef5f694c8" />


Posteriormente, se realizó la configuración de puertos en el adaptador NAT.

### Configuración de dirección IP

Se configuró la dirección IP en el archivo YAML correspondiente.
<img width="345" height="353" alt="image" src="https://github.com/user-attachments/assets/383474d8-3afd-4c81-bc74-bbf48330e673" />


Después de ello, se ejecutó el siguiente comando para verificar que no existieran errores y aplicar la configuración:

```bash
sudo netplan try
```

Finalmente, mediante:

```bash
ip a
```

se comprobó que la dirección IP del adaptador interno fue modificada correctamente.
<img width="886" height="522" alt="image" src="https://github.com/user-attachments/assets/27604af2-bdba-455d-8573-58f80456de9d" />

---

## 2. Habilitación del reenvío de paquetes

Se eliminó el símbolo `#` de la línea correspondiente al **IP forwarding** dentro del archivo:

```bash
/etc/sysctl.conf
```
<img width="886" height="522" alt="image" src="https://github.com/user-attachments/assets/17137479-9e60-43e9-b80d-4087b805157f" />

Con esta configuración se permitió la salida a internet desde las demás máquinas virtuales.

Posteriormente, se configuraron las reglas necesarias en **iptables** para permitir el enrutamiento adecuado de tráfico interno hacia internet.
<img width="841" height="317" alt="image" src="https://github.com/user-attachments/assets/433f7b74-e8ea-4a69-bdeb-a560c013d1ea" />


---

## 3. Instalación y configuración de Bind9

Se realizó la instalación del servicio **Bind9** para la administración de dominios DNS.
<img width="742" height="139" alt="image" src="https://github.com/user-attachments/assets/c02001ac-0a35-4a8b-95a5-629cc4fe7014" />


Luego, se agregó el dominio y la ruta de configuración correspondiente en el archivo:

```bash
/etc/bind/named.conf.local
```
<img width="425" height="120" alt="image" src="https://github.com/user-attachments/assets/6f04d5e6-9c7f-4e9e-b7c6-b8713367f161" />

Posteriormente, se realizó una copia del archivo de configuración por defecto para crear una configuración personalizada.

<img width="747" height="47" alt="image" src="https://github.com/user-attachments/assets/bf15e07d-697b-418c-85c4-966df475af43" />

<img width="713" height="375" alt="image" src="https://github.com/user-attachments/assets/2dd665ba-7537-41b0-8d24-abc396577e43" />

Para validar que no existieran errores de sintaxis se verificó el archivo configurado.

<img width="655" height="88" alt="image" src="https://github.com/user-attachments/assets/147f59f9-6a14-4b40-b29d-b15acff3ab2c" />

Finalmente, se reinició el servicio Bind9 para aplicar los cambios:

```bash
sudo systemctl restart bind9
```

<img width="886" height="501" alt="image" src="https://github.com/user-attachments/assets/0a2b0e52-b3b2-4963-ae47-cbab4a4d0880" />


Después se verificó que el servidor estuviera escuchando peticiones mediante el puerto **53**.


---<img width="886" height="76" alt="image" src="https://github.com/user-attachments/assets/57f9dec7-5647-45cb-9b8d-164595da0039" />


## 4. Configuración del servidor web

Se asignó un adaptador interno al servidor web y se configuró su dirección IP mediante el archivo YAML.

<img width="381" height="378" alt="image" src="https://github.com/user-attachments/assets/6c3849a9-0e0f-4f4d-9a59-bce852093880" />

Una vez aplicada la configuración, se realizaron pruebas de conectividad con el gateway.

<img width="800" height="225" alt="image" src="https://github.com/user-attachments/assets/7cbdeb6a-aa63-466b-8594-eb0936272947" />

También se verificó que la salida a internet fuera exitosa.

<img width="794" height="231" alt="image" src="https://github.com/user-attachments/assets/6ac2913b-c3a4-40cb-b110-9ae33286b2ce" />

### Instalación de NGINX

Se instaló y configuró el servidor web **NGINX**.

<img width="886" height="54" alt="image" src="https://github.com/user-attachments/assets/0082df87-a38a-41e5-bc5e-cca0f39892a0" />

Posteriormente, se creó un nuevo bloque de configuración dentro del archivo:

```bash
/etc/nginx/sites-available/lab42.local
```

<img width="563" height="281" alt="image" src="https://github.com/user-attachments/assets/c7b054b8-9e22-4870-b86f-90646bd14438" />

Luego:

- Se activó el sitio.
- Se creó el contenido que se visualizaría en la página web.
- Se verificó la correcta configuración de los archivos.
- Se reinició NGINX para aplicar los cambios.

<img width="886" height="78" alt="image" src="https://github.com/user-attachments/assets/601e737a-015a-4105-b2c5-318c36aea0b7" />

<img width="847" height="102" alt="image" src="https://github.com/user-attachments/assets/d61908eb-997e-407a-afce-6e768c111dbf" />

---

## 5. Configuración de la máquina cliente

Se configuró la dirección IP de la máquina virtual cliente.

<img width="378" height="402" alt="image" src="https://github.com/user-attachments/assets/a58de97c-d402-4c95-b2dc-aedfaa522e01" />

Posteriormente, se realizaron pruebas de resolución DNS mediante:

```bash
dig
```

<img width="886" height="711" alt="image" src="https://github.com/user-attachments/assets/8a5a604f-ce7d-445d-afe9-ca1faf4502b3" />

y

```bash
nslookup
```

Los resultados mostraron que la resolución del dominio se realizó correctamente.

Asimismo, se comprobó que la página web resolvía adecuadamente el nombre configurado.

<img width="569" height="56" alt="image" src="https://github.com/user-attachments/assets/887c3629-f7fc-412e-8775-8530940fa538" />

---

## 6. Parte práctica en grupo

### Configuración DNS grupal

Para esta práctica, el servidor DNS utilizó la dirección IP:

```text
10.140.170.200
```

con el dominio:

```text
los-pepes.red
```

<img width="369" height="352" alt="image" src="https://github.com/user-attachments/assets/693dcd4c-9367-42cc-ba5f-79b99aae3954" />

Se realizó la configuración de Bind9 creando dos archivos:

- Uno para la resolución directa del dominio.
- Otro para la resolución inversa mediante dirección IP.

<img width="488" height="234" alt="image" src="https://github.com/user-attachments/assets/24e032ef-347c-4f2f-9d74-cb1c061f5f0c" />

<img width="820" height="323" alt="image" src="https://github.com/user-attachments/assets/85616541-36ad-4fb0-be1f-30f2ddfbecbd" />

<img width="841" height="300" alt="image" src="https://github.com/user-attachments/assets/4e799139-bf77-44c9-a50a-b2fcb0f49742" />

<img width="688" height="208" alt="image" src="https://github.com/user-attachments/assets/e4dcd338-946c-4af5-94a9-8393124a206d" />



Finalmente, se realizaron verificaciones mediante consultas DNS, comprobando el correcto funcionamiento del dominio configurado.

---
## 2. Configuración Inicial de Red del Servidor Web

Como primer paso se configuró manualmente la interfaz de red del
servidor web mediante Netplan, asignando la IP estática correspondiente
dentro del segmento proporcionado para el grupo.
<img width="962" height="196" alt="22  ping web - dns (grupal)" src="https://github.com/user-attachments/assets/2961e680-87ae-40d2-825f-44be4b97fef0" />
<img width="961" height="401" alt="20  Configuración yaml web (grupal)" src="https://github.com/user-attachments/assets/d343b769-7e5f-4dbe-aaab-4c84a8d890bc" />

La dirección asignada al servidor web fue:

`10.140.170.201/24`

El gateway y servidor DNS utilizado fue:

`10.140.170.200`

Para ello se modificó el archivo `/etc/netplan/50-cloud-init.yaml`,
donde se estableció que la interfaz `enp0s3` trabajara sin DHCP,
utilizando direccionamiento manual, nameserver y ruta por defecto hacia
el DNS. Posteriormente se aplicaron los cambios para activar la nueva
configuración.

Tras esto se verificó exitosamente mediante `ip a` que la máquina
mostrara correctamente la IP asignada, confirmando así que el servidor
web ya pertenecía correctamente a la red grupal.

------------------------------------------------------------------------

## 3. Validación de Conectividad con el Servidor DNS

Una vez configurada la red, se realizaron pruebas de conectividad usando
`ping` hacia la IP del compañero responsable del DNS (`10.140.170.200`).

<img width="962" height="196" alt="22  ping web - dns (grupal)" src="https://github.com/user-attachments/assets/9ec839d0-303c-475f-959a-e54762307559" />


Los resultados fueron exitosos, mostrando 0% de pérdida de paquetes y
tiempos de respuesta estables, confirmando que ambas máquinas podían
comunicarse correctamente dentro de la red.

Esta validación fue esencial, ya que garantizó que el servidor web
pudiera depender del DNS para resolución de nombres y acceso grupal.

------------------------------------------------------------------------

## 4. Instalación del Servidor Nginx

Con la conectividad validada, se procedió a instalar Nginx como servicio
web principal.

Durante el primer intento se presentó un bloqueo temporal del sistema
debido a procesos automáticos de actualización (`unattended-upgrades`),
impidiendo la instalación inmediata. Luego de esperar la liberación del
bloqueo, se ejecutó nuevamente la instalación, logrando completar
exitosamente la descarga, desempaquetado e instalación del paquete Nginx
junto con `nginx-common`.

Posteriormente se verificó que Nginx ya estuviera instalado
correctamente y sin errores adicionales.

Una vez instalado, se habilitó el servicio con inicio automático
utilizando `systemctl`, asegurando que Nginx se ejecutara desde el
arranque del sistema operativo.

------------------------------------------------------------------------

## 5. Configuración del Virtual Host

Después de instalar Nginx, se creó la configuración personalizada del
sitio web en `/etc/nginx/sites-available/lab42.local`.

Aunque inicialmente el laboratorio individual trabajó con `lab42.local`,
para la práctica grupal se adaptó el funcionamiento al dominio real
asignado:

`los-pepes.red`

Dentro de la configuración del servidor se estableció:

-   Escucha en puerto 80\
-   `server_name` para dominio principal y `www`\
-   Directorio raíz `/var/www/lab42.local`\
-   Archivo principal `index.html`\
-   Validación de recursos mediante `try_files`

Esta estructura permitió que Nginx respondiera correctamente a
solicitudes realizadas tanto por IP como por nombre de dominio.

Luego se verificó la sintaxis de Nginx mediante `sudo nginx -t`,
obteniendo como resultado:
<img width="969" height="80" alt="24  nginx -t (grupal)" src="https://github.com/user-attachments/assets/6fadf206-8e93-4b16-80aa-fb6a5a454d95" />


`syntax is ok`\
`test is successful`

Esto confirmó que no existían errores en la configuración antes de
reiniciar el servicio.

------------------------------------------------------------------------

## 6. Integración con DNS y Resolución de Dominio

Con Nginx operativo, se procedió a validar la correcta resolución DNS
usando:

`nslookup www.los-pepes.red 10.140.170.200`

El resultado devolvió exitosamente:

`www.los-pepes.red -> 10.140.170.201`

Esto confirmó que el servidor DNS del compañero estaba enlazando
correctamente el dominio grupal hacia el servidor web configurado.

Esta fase fue clave, ya que permitió vincular infraestructura DNS + Web
en un entorno funcional completo.

------------------------------------------------------------------------

## 7. Validación desde Navegador

Finalmente, se realizaron pruebas desde navegador utilizando:

-   `http://10.140.170.201`
-   `http://los-pepes.red`

En ambos casos el sistema mostró correctamente la página configurada,
desplegando el mensaje:

<img width="960" height="334" alt="25  navegador funcionando (grupal)" src="https://github.com/user-attachments/assets/94b8e551-516c-4d26-b914-6c6393bba0ba" />


**Bienvenido a www.los-pepes.red**\
**Servidor Web funcionando correctamente**\
**Web: Adalid Gutiérrez Torricos**\
**DNS: Dylan Huayta Fuertes**

Esto confirmó que:

-   Nginx servía correctamente el contenido HTML\
-   El dominio resolvía adecuadamente\
-   La integración DNS-Web era completamente funcional\
-   El laboratorio grupal fue completado con éxito

## 7. Conclusiones

Durante el desarrollo de este laboratorio se logró implementar correctamente un entorno funcional de servicios de red, integrando:

- Configuración de servidor DNS
- Enrutamiento hacia internet
- Servidor web con NGINX
- Resolución de nombres desde cliente

Se comprobó la correcta interacción entre todos los componentes mediante pruebas prácticas de conectividad y resolución de dominios.

Esta práctica permitió reforzar conocimientos sobre administración de redes, configuración de servicios y resolución de problemas en entornos virtualizados.
