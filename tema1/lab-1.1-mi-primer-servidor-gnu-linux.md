# Laboratorio 1.1: Mi Primer Servidor GNU/Linux

**Universidad San Francisco Xavier de Chuquisaca**

**Asignatura:** Infraestructura, Plataformas Tecnológicas y Redes (SIS313)

**Docente:** Ing. Marcelo Quispe Ortega

**Semestre:** 1/2026

## Introducción para el Estudiante

En Linux, no usamos ventanas ni ratón para administrar servidores. Usamos la **Terminal**. Cada comando es una instrucción precisa. La terminal distingue entre mayúsculas y minúsculas (`Archivo.txt` no es lo mismo que `archivo.txt`).

## 🎯 Objetivo del Laboratorio


## 🛠️ Sección 1: Preparación del Entorno Virtual

Para este laboratorio, cada estudiante debe tener una máquina virtual (VM) limpia.

1. **Crear una nueva VM:** En VirtualBox, crear una máquina llamada con las siguientes características:
    - En la sección "Nombre y Sistema Operativo":
        - Nombre: SIS313-Lab1.1
        - Descarga y selecciona el ISO [Ubuntu Server 24.04.4 LTS](https://ubuntu.com/download/server). Verifica que se marque por defecto:
            - Tipo: Linux
            - Subtipo: Ubuntu
            - Versión: Ubuntu (64-bit)
        - Marca la opción "Omitir instalación desatendida".
    - En la sección "Hardware":
        - Selecciona 1024 MB si tienes 4GB de RAM en tu PC o 2048 MB si tienes 6GB o superior.
        - En Procesadores, selecciona 1 CPU si tienes 4 CPUs  y 2 CPU si tienes superior a $ CPUs.
    - Y por último en la sección "Disco Duro":
        - Crea un disco virtual con 20 GB de capacidad.


2. **Personalización de la VM:**
    - Explora las opciones de la VM seleccionandola y haciendo clic en Configuración.
    - Dentro de Almacenamiento añade un segundo disco con 5GB de capacidad dentro de los controladores SATA.
    - Acepta los cambios para guardar la nueva configuración.

3. **Encender e Instalar:** Inicia la VM e instala Ubuntu Server considerando lo siguiente:

    - Escoger idioma Español para la instalación
    - Selecciona la distribución del teclado de acuerdo al teclado que tengas en tu PC.
    - Configura la red por defecto.
    - No configures el proxy y dejalo vacio por defecto.
    - Usa el disco entero para la instalación, solo toma en cuenta instalar el SO en el disco de 20 GB, el otro no es necesario configurarlo en este paso.
    - En la Configuración de perfil, introduce tus datos de usuario.
    - En Configuración de SSH, instala el servidor OpenSSH
    - No selecciones ningún software adicional
    - Al finalizar, iniciar sesión con su usuario y contraseña.


## 💻 Sección 2: Práctica guiada


## ⚙️ Sección 3: Práctica  Individual


### ✅ Evaluación del Laboratorio