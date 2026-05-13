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

Posteriormente, se realizó la configuración de puertos en el adaptador NAT.

### Configuración de dirección IP

Se configuró la dirección IP en el archivo YAML correspondiente.

Después de ello, se ejecutó el siguiente comando para verificar que no existieran errores y aplicar la configuración:

```bash
sudo netplan try
```

Finalmente, mediante:

```bash
ip a
```

se comprobó que la dirección IP del adaptador interno fue modificada correctamente.

---

## 2. Habilitación del reenvío de paquetes

Se eliminó el símbolo `#` de la línea correspondiente al **IP forwarding** dentro del archivo:

```bash
/etc/sysctl.conf
```

Con esta configuración se permitió la salida a internet desde las demás máquinas virtuales.

Posteriormente, se configuraron las reglas necesarias en **iptables** para permitir el enrutamiento adecuado de tráfico interno hacia internet.

---

## 3. Instalación y configuración de Bind9

Se realizó la instalación del servicio **Bind9** para la administración de dominios DNS.

Luego, se agregó el dominio y la ruta de configuración correspondiente en el archivo:

```bash
/etc/bind/named.conf.local
```

Posteriormente, se realizó una copia del archivo de configuración por defecto para crear una configuración personalizada.

Para validar que no existieran errores de sintaxis se verificó el archivo configurado.

Finalmente, se reinició el servicio Bind9 para aplicar los cambios:

```bash
sudo systemctl restart bind9
```

Después se verificó que el servidor estuviera escuchando peticiones mediante el puerto **53**.

---

## 4. Configuración del servidor web

Se asignó un adaptador interno al servidor web y se configuró su dirección IP mediante el archivo YAML.

Una vez aplicada la configuración, se realizaron pruebas de conectividad con el gateway.

También se verificó que la salida a internet fuera exitosa.

### Instalación de NGINX

Se instaló y configuró el servidor web **NGINX**.

Posteriormente, se creó un nuevo bloque de configuración dentro del archivo:

```bash
/etc/nginx/sites-available/lab42.local
```

Luego:

- Se activó el sitio.
- Se creó el contenido que se visualizaría en la página web.
- Se verificó la correcta configuración de los archivos.
- Se reinició NGINX para aplicar los cambios.

---

## 5. Configuración de la máquina cliente

Se configuró la dirección IP de la máquina virtual cliente.

Posteriormente, se realizaron pruebas de resolución DNS mediante:

```bash
dig
```

y

```bash
nslookup
```

Los resultados mostraron que la resolución del dominio se realizó correctamente.

Asimismo, se comprobó que la página web resolvía adecuadamente el nombre configurado.

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

Se realizó la configuración de Bind9 creando dos archivos:

- Uno para la resolución directa del dominio.
- Otro para la resolución inversa mediante dirección IP.

Finalmente, se realizaron verificaciones mediante consultas DNS, comprobando el correcto funcionamiento del dominio configurado.

---

## 7. Conclusiones

Durante el desarrollo de este laboratorio se logró implementar correctamente un entorno funcional de servicios de red, integrando:

- Configuración de servidor DNS
- Enrutamiento hacia internet
- Servidor web con NGINX
- Resolución de nombres desde cliente

Se comprobó la correcta interacción entre todos los componentes mediante pruebas prácticas de conectividad y resolución de dominios.

Esta práctica permitió reforzar conocimientos sobre administración de redes, configuración de servicios y resolución de problemas en entornos virtualizados.
