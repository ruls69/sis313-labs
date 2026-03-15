# Laboratorio 1.1: Mi Primer Servidor GNU/Linux

**Universidad San Francisco Xavier de Chuquisaca**

**Asignatura:** Infraestructura, Plataformas Tecnológicas y Redes (SIS313)

**Docente:** Ing. Marcelo Quispe Ortega

**Semestre:** 1/2026

## Introducción para el Estudiante

En Linux, no usamos ventanas ni ratón para administrar servidores. Usamos la **Terminal**. Cada comando es una instrucción precisa. La terminal distingue entre mayúsculas y minúsculas (`Archivo.txt` no es lo mismo que `archivo.txt`).

## 🎯 Objetivo del Laboratorio


## 🛠️ Sección 1: Preparación del Entorno Virtual

En esta etapa, simularemos la **provisión de hardware**. En un entorno real, estarías instalando un servidor físico en un rack; aquí, utilizaremos **VirtualBox** para crear un "servidor virtual" con las especificaciones necesarias para nuestra infraestructura.

**1.1. Creación de la Máquina Virtual (VM)**

Abre VirtualBox y haz clic en el botón **Nueva**. Sigue estas configuraciones cuidadosamente:

- **Nombre y Sistema Operativo:**

    - **Nombre:** `SIS313-Lab1.1`

    - **Imagen ISO:** Busca y selecciona el archivo `.iso` de **Ubuntu Server 24.04.4 LTS** que [descargaste](https://ubuntu.com/download/server).
    
    - **Tipo y Versión:** Verifica que se auto-seleccionen como **Linux** y **Ubuntu (64-bit)**.

    - **Importante:** Marca la casilla **Omitir instalación desatendida**. Esto es vital para que tú mismo realices el proceso de instalación y aprendas cada paso.

    - **Hardware (Recursos del sistema):**

        Ajusta los recursos según la potencia de tu computadora física:

        - **Memoria Base (RAM):**

            - Si tu PC tiene 4 GB de RAM: Asigna **1024 MB**.

            - Si tu PC tiene 6 GB o más: Asigna **2048 MB**.

        - **Procesadores:**

            - Si tu PC tiene hasta 4 núcleos: Asigna **1 CPU**.

            - Si tu PC tiene más de 4 núcleos: Asigna **2 CPUs**.

        - **Disco Duro Virtual:**

            - Selecciona "Crear un disco duro virtual ahora".

            - Tamaño del disco: **20.00 GB**.


**1.2. Simulación de Almacenamiento Adicional**

Un servidor de infraestructura suele tener varios discos para separar el sistema de los datos. Vamos a añadir un segundo "disco físico":

1. Con la VM seleccionada, haz clic en el botón naranja de **Configuración**.

2. Ve al apartado de **Almacenamiento** en el menú de la izquierda.

3. En el árbol de almacenamiento, selecciona el **Controlador: SATA**.

4. Haz clic en el icono de **Añadir Disco Duro** (el icono de disco pequeño con un signo + verde).

5. Elige **Crear** y sigue el asistente (VDI, Reservado dinámicamente) para crear un nuevo disco de **5.00 GB**.

6. Al finalizar, deberías ver dos archivos .vdi bajo el controlador SATA: uno de 20 GB y otro de 5 GB. Haz clic en **Aceptar**.

**1.3. Instalación del Sistema Operativo**

Inicia la VM haciendo clic en **Flecha Verde (Iniciar)**. El instalador de Ubuntu es puramente textual; usa las flechas del teclado para moverte y la tecla **Enter** para seleccionar.

1. **Idioma:** Selecciona **Spanish** (o Español).

2. **Teclado:** Elige la distribución que coincida con tu teclado físico (usualmente *Spanish* o *Spanish - Latin American*). Puedes probarlo en la sección "Identify keyboard".

3. **Tipo de instalación:** Selecciona la opción por defecto: **Ubuntu Server**.

4. **Red:** El instalador detectará una IP automáticamente (DHCP). No realices cambios, presiona **Hecho**.

5. **Proxy y Mirror:** Deja ambos campos en blanco (por defecto) y presiona **Hecho**.

6. **Configuración de Almacenamiento:**

    - Selecciona **Use an entire disk** (Usar todo el disco).

    - **¡Cuidado!** Asegúrate de que el disco seleccionado sea el de **20 GB**.

    - Presiona **Hecho** y confirma la escritura en el disco cuando se te solicite.

7. **Configuración del Perfil:** Introduce tu nombre, el nombre del servidor (ej. `srv-lab1`), tu nombre de usuario y una contraseña que no olvides (sugerencia: tu documento de identidad).

8. **Upgrade a Ubuntu Pro:** Selecciona **Skip for now** (Omitir por ahora).

9. **Configuración de SSH:** Marca con la barra espaciadora la opción **Install OpenSSH server**. Esto nos permitirá administrar el servidor remotamente más adelante.

10. **Software adicional:** No selecciones nada de la lista. Presiona **Hecho**.

11. **Finalización:** El sistema comenzará a instalarse. Cuando termine, aparecerá la opción **Reboot Now**. Presiona Enter, retira el medio de instalación si VirtualBox te lo pide, y espera a que aparezca el prompt de login.

**1.4. Acceso Remoto: Configuración de Reenvío de Puertos (Port Forwarding)**

Como tu servidor está dentro de una red interna de VirtualBox (NAT), tu computadora física no "ve" directamente al servidor. Para administrarlo de forma profesional, vamos a crear un "puente" o túnel que conecte un puerto de tu PC con el puerto de servicio SSH del servidor.

**A. Configuración en VirtualBox**

1. Con la VM **SIS313-Lab1.1** seleccionada (puede estar encendida o apagada), haz clic en **Configuración**.

2. Ve al menú **Red**.

3. Asegúrate de que en el "Adaptador 1" esté seleccionado **Conectado a: NAT**.

4. Haz clic en el botón desplegable **Avanzadas**.

5. Haz clic en el botón **Reenvío de puertos**.

6. En la ventana que aparece, haz clic en el icono de **Agregar nueva regla** (el símbolo `+` verde a la derecha) y completa los datos exactamente así:

    | Nombre | Protocolo | IP anfitrión | Puerto anfitrión | IP invitado | Puerto invitado |
    | - | - | - | - | - | - |
    | SSH | TCP |   | 2222 | 10.0.2.15 | 22 |

    **¿Qué significa esto?** Le estamos diciendo a VirtualBox: "*Cualquier petición que llegue a mi PC real por el puerto 2222, envíala automáticamente al puerto 22 (SSH) de mi servidor virtual*".

7. Haz clic en **Aceptar** en ambas ventanas para guardar.

**B. Instalación de la Terminal y Prueba de Conexión**

Para administrar servidores, los profesionales no suelen usar la ventana pequeña de VirtualBox, sino una terminal moderna.

1. **Instala Warp:** Descarga e instala [Warp Terminal](https://app.warp.dev/referral/3DY6RJ). Es una terminal inteligente que te ayudará mucho en este semestre.

2. **Prueba la conexión:** Abre Warp en tu PC física y escribe el siguiente comando (reemplaza `marcelo` por el nombre de usuario que elegiste durante la instalación):

    ``` bash
    ssh -p 2222 marcelo@127.0.0.1
    ```
3. **Primer acceso:** La terminal te preguntará: *"Are you sure you want to continue connecting (yes/no/[fingerprint])?"*. Escribe **yes** y presiona **Enter**.

4. **Contraseña:** Introduce la contraseña que configuraste en la instalación. **Nota:** No verás asteriscos ni puntos mientras escribes por seguridad; solo escribe y presiona **Enter**.

Si lograste entrar, verás que el texto de la terminal cambia (ej. `marcelo@srv-lab1:~$`). ¡Felicidades, ya estás administrando tu servidor de forma remota!

## 💻 Sección 2: Práctica guiada


## ⚙️ Sección 3: Práctica  Individual


### ✅ Evaluación del Laboratorio