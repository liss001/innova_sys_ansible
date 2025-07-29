# Proyecto de Configuración de Servidor InnovaSys con Ansible

Este repositorio contiene un playbook de Ansible diseñado para automatizar la configuración de un servidor Ubuntu Server 24.04 (nodo gestionado) para cumplir con los requisitos de la startup "InnovaSys": un servidor web (Apache) y un servidor de archivos (Samba). El objetivo es establecer un entorno de intranet funcional y un recurso compartido de archivos.

---

## Requisitos del Laboratorio

* **Nodo de Control:** Linux Lite (con Ansible instalado).
    * Conectividad a Internet (para descargar paquetes).
    * Configuración SSH para acceder al nodo gestionado (clave SSH o contraseña).
* **Nodo Gestionado:** Ubuntu Server 24.04 (servidor-innovasys).
    * Usuario con privilegios `sudo` configurado (ej. `operador`).
* **Conectividad de Red:**
    * Las máquinas virtuales deben estar configuradas en VirtualBox con al menos una "Red Interna" (`intranet-innovasys`) que permita la comunicación SSH entre el nodo de control y el gestionado.
    * El nodo de control (Linux Lite) también debe tener un adaptador de red en modo **NAT** para acceder a Internet (para GitHub, actualizaciones, etc.).

---

## Estructura del Proyecto

El proyecto está organizado usando roles de Ansible para una mejor modularidad y buenas prácticas:

innova_sys_ansible/
├── playbook.yml            # Playbook principal que orquesta los roles.
├── inventory.ini           # Archivo de inventario con la IP del servidor gestionado.
├── README.md               # Este archivo de documentación.
└── roles/
├── apache/             # Rol para la configuración de Apache.
│   ├── tasks/
│   │   └── main.yml    # Tareas de instalación y configuración de Apache.
│   ├── templates/
│   │   └── index.html.j2 # Plantilla Jinja2 para la página web personalizada.
│   └── handlers/
│       └── main.yml    # Handlers para reiniciar Apache.
└── samba/              # Rol para la configuración de Samba.
├── tasks/
│   └── main.yml    # Tareas de instalación y configuración de Samba.
├── templates/
│   └── smb.conf.j2 # (Plantilla para smb.conf si usas Jinja2)
└── handlers/
└── main.yml    # Handlers para reiniciar Samba.


---

## Cómo Ejecutar el Playbook

Sigue estos pasos desde tu **Nodo de Control (Linux Lite)** para configurar el `servidor-innovasys`:

1.  **Clonar el Repositorio (si estás en una máquina diferente):**
    Si estás en una máquina diferente al Linux Lite original y necesitas el código, clona este repositorio:
    ```bash
    git clone [https://github.com/liss001/innova_sys_ansible.git](https://github.com/liss001/innova_sys_ansible.git)
    cd innova_sys_ansible
    ```
    *Asegúrate de reemplazar la URL con la de tu propio repositorio si la cambiaste en GitHub.*

2.  **Verificar el Inventario:**
    Asegúrate de que el archivo `inventory.ini` apunte correctamente a tu `servidor-innovasys`. Debe contener algo similar a:
    ```ini
    [servidores_web]
    servidor-innovasys ansible_host=192.168.10.100 ansible_user=operador ansible_ssh_private_key_file=~/.ssh/id_rsa

    [all:vars]
    ansible_python_interpreter=/usr/bin/python3
    nombre_empresa="InnovaSys"
    ```
    * **Importante:** Asegúrate de que `ansible_host` sea la IP correcta de tu `servidor-innovasys` en la red interna.
    * Si usas autenticación por contraseña SSH en lugar de claves SSH, ajusta el inventario en consecuencia (ej: `ansible_user=operador ansible_password=tu_contraseña_ssh`).

3.  **Ejecutar el Playbook:**
    Desde la raíz del proyecto (`innova_sys_ansible/`), ejecuta el playbook:
    ```bash
    ansible-playbook -i inventory.ini playbook.yml --ask-become-pass
    ```
    * Se te pedirá la `BECOME password` (la contraseña de `sudo`) de tu usuario `operador` en `servidor-innovasys`.

---

## Verificación Post-Configuración

Una vez que el playbook haya finalizado con `ok` y `changed` (sin `failed` o `unreachable`), puedes verificar la configuración:

1.  **Servidor Web (Apache):**
    Abre un navegador web en tu Linux Lite y navega a:
    ```
    [http://192.168.10.100](http://192.168.10.100)
    ```
    Deberías ver la página de bienvenida personalizada de InnovaSys, como "¡Bienvenidos a la Intranet de InnovaSys!".

2.  **Servidor de Archivos (Samba):**
    Desde el gestor de archivos de Linux Lite, conéctate al recurso compartido:
    ```
    smb://192.168.10.100/Proyectos
    ```
    Cuando se te soliciten credenciales, usa:
    * **Usuario:** `devuser1`
    * **Contraseña:** `Innova.2025`
    Deberías poder acceder al directorio y **crear un nuevo archivo** dentro de él, confirmando que el acceso es escribible y limitado a los miembros del grupo `desarrolladores`.
