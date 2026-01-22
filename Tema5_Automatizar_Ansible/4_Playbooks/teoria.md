# 4. Playbooks en Ansible

## ¿Qué es un Playbook?

Un **playbook** es un archivo en formato YAML que describe un conjunto de tareas a ejecutar sobre uno o varios hosts. Es la unidad principal de automatización en Ansible y permite definir configuraciones, despliegues y orquestación de servicios de forma estructurada, repetible y declarativa.

Los playbooks representan **"recetas"** o **"planes de acción"** que describen el estado deseado de los sistemas.

## Características de los Playbooks

- **Legibles:** Escritos en YAML, fáciles de leer y entender
- **Declarativos:** Describes QUÉ quieres, no CÓMO hacerlo
- **Idempotentes:** Se pueden ejecutar múltiples veces sin efectos secundarios
- **Reutilizables:** Se pueden versionar y compartir
- **Modulares:** Permiten incluir roles y otros playbooks
- **Potentes:** Admiten variables, condicionales, bucles, y más

## Sintaxis y estructura de un Playbook

Los playbooks siguen esta estructura básica:

```yaml
---
# Inicio del documento YAML

- name: Nombre descriptivo del play
  hosts: grupo_o_host              # Hosts objetivo
  gather_facts: yes                # Recopilar información (por defecto)
  become: yes                      # Ejecutar como root (opcional)
  vars:                            # Variables (opcional)
    variable1: valor1
  tasks:                           # Lista de tareas
    - name: Descripción tarea 1
      módulo:
        parámetro: valor
    - name: Descripción tarea 2
      módulo:
        parámetro: valor
      notify: Nombre handler        # Notificar handler (opcional)
  handlers:                        # Handlers (opcional)
    - name: Nombre handler
      módulo:
        parámetro: valor
  roles:                           # Roles (opcional)
    - nombre_rol
```

### Componentes de un Play

Un playbook está compuesto por una lista de "**plays**". Cada play define:

1. **hosts:** Los hosts o grupos sobre los que se ejecutarán las tareas
2. **vars:** Variables específicas para el play
3. **tasks:** Lista de acciones a realizar (instalar paquetes, copiar archivos, etc)
4. **handlers:** Tareas que se ejecutan solo cuando son notificadas
5. **roles:** Permite incluir roles para organizar la configuración

## Ejemplo básico de Playbook

```yaml
---
- name: Instalar y arrancar Apache
  hosts: webservers
  become: yes
  tasks:
    - name: Instalar Apache
      apt:
        name: apache2
        state: present
    
    - name: Asegurar que Apache esté iniciado
      service:
        name: apache2
        state: started
        enabled: yes
```

**Ejecución:**
```bash
ansible-playbook apache.yml
```

**Salida esperada:**
```
PLAY [Instalar y arrancar Apache] **************************************

TASK [Gathering Facts] *************************************************
ok: [web1]

TASK [Instalar Apache] *************************************************
changed: [web1]

TASK [Asegurar que Apache esté iniciado] *******************************
changed: [web1]

PLAY RECAP *************************************************************
web1                       : ok=3    changed=2    unreachable=0    failed=0
```

## Ejemplo con Variables y Handlers

```yaml
---
- name: Desplegar aplicación web
  hosts: webservers
  become: yes
  vars:
    paquete: apache2
    puerto: 80
    mensaje: "Servidor web funcionando"
  
  tasks:
    - name: Instalar {{ paquete }}
      apt:
        name: "{{ paquete }}"
        state: present
        update_cache: yes
    
    - name: Crear página personalizada
      copy:
        content: "<h1>{{ mensaje }}</h1>"
        dest: /var/www/html/index.html
      notify: Reiniciar Apache
    
    - name: Asegurar que Apache esté iniciado
      service:
        name: "{{ paquete }}"
        state: started
        enabled: yes
  
  handlers:
    - name: Reiniciar Apache
      service:
        name: "{{ paquete }}"
        state: restarted
```

## Gathering Facts

De forma predeterminada, Ansible **recopila información** (facts) de los hosts antes de ejecutar las tareas. Esta información incluye:

- Nombre del host
- Sistema operativo
- Dirección IP
- Memoria y CPU
- Variables de entorno
- Y mucha más información

**Ejemplo de uso de facts:**
```yaml
---
- name: Mostrar información del sistema
  hosts: all
  gather_facts: true
  tasks:
    - name: Mostrar información
      debug:
        msg: |
          Hostname: {{ ansible_hostname }}
          OS: {{ ansible_distribution }} {{ ansible_distribution_version }}
          IP: {{ ansible_default_ipv4.address }}
          CPU: {{ ansible_processor_cores }}
```

**Desactivar gathering facts:**
```yaml
---
- name: Playbook rápido
  hosts: all
  gather_facts: false    # Más rápido si no necesitas la información
  tasks:
    - name: Mostrar mensaje
      debug:
        msg: "Hola Ansible"
```

## Playbooks con múltiples Plays

Un mismo archivo puede contener múltiples plays para diferentes grupos de hosts:

```yaml
---
# Play 1: Configurar servidores web
- name: Configurar servidores web
  hosts: webservers
  become: yes
  tasks:
    - name: Instalar Apache
      apt:
        name: apache2
        state: present

# Play 2: Configurar bases de datos
- name: Configurar servidores de BD
  hosts: databases
  become: yes
  tasks:
    - name: Instalar MySQL
      apt:
        name: mysql-server
        state: present

# Play 3: Configurar todos los servidores
- name: Configuración común
  hosts: all
  become: yes
  tasks:
    - name: Actualizar sistema
      apt:
        upgrade: dist
        update_cache: yes
```

## Operaciones para Playbooks sencillos

A continuación se muestran ejemplos de operaciones comunes con módulos de Ansible.

### 1. Modificación de archivos con blockinfile

El módulo **blockinfile** inserta, actualiza o elimina un bloque de líneas en un archivo. El texto modificado queda delimitado por líneas que actúan como marcador.

**Ejemplo:**
```yaml
---
- name: Modificar /etc/hosts
  hosts: all
  become: yes
  tasks:
    - name: "Añadir entradas al archivo /etc/hosts"
      blockinfile:
        path: /etc/hosts
        block: |
          # Manager
          20.0.1.7 manager

          # Managed-1
          20.0.1.11 managed-1

          # Managed-2
          20.0.1.4 managed-2
        marker: "# {mark} ANSIBLE MANAGED BLOCK"
```

**Resultado en /etc/hosts:**
```
127.0.0.1 localhost
...
# BEGIN ANSIBLE MANAGED BLOCK
# Manager
20.0.1.7 manager

# Managed-1
20.0.1.11 managed-1

# Managed-2
20.0.1.4 managed-2
# END ANSIBLE MANAGED BLOCK
```

### 2. Modificación de archivos con lineinfile

El módulo **lineinfile** asegura que exista una línea con un texto concreto en un archivo. Usa expresiones regulares para la búsqueda.

**Ejemplo - Solucionar mensaje "unable to resolve host":**
```yaml
---
- name: Añadir hostname a /etc/hosts
  hosts: all
  become: yes
  tasks:
    - name: "Añadir hostname a /etc/hosts"
      lineinfile:
        path: /etc/hosts
        insertafter: '^127\.0\.0\.1'           # Buscar línea que empieza por 127.0.0.1
        line: "127.0.0.1 {{ ansible_hostname }}"  # Insertar después
```

### 3. Uso de templates para crear archivos personalizados

El módulo **template** permite incluir archivos en los nodos administrados sustituyendo variables.

**Archivo template (sample-template.txt.j2):**
```
Ejemplo de archivo personalizado usando templates:

El nodo {{ manager.name }} tiene la IP: {{ manager.ip }}.
El nodo {{ managed_1.name }} tiene la IP: {{ managed_1.ip }}.
El nodo {{ managed_2.name }} tiene la IP: {{ managed_2.ip }}.
```

**Playbook:**
```yaml
---
- name: Usar templates
  hosts: all
  become: yes
  vars:
    manager: { name: manager, ip: 20.0.1.7 }
    managed_1: { name: managed-1, ip: 20.0.1.11 }
    managed_2: { name: managed-2, ip: 20.0.1.4 }
  tasks:
    - name: "Crear archivo personalizado"
      template:
        src: sample-template.txt.j2
        dest: /home/ubuntu/sample-template.txt
        owner: ubuntu
        group: ubuntu
        mode: '0644'
```

### 4. Gestión de repositorios APT

El módulo **apt_repository** permite añadir o eliminar repositorios APT.

**Añadir repositorio:**
```yaml
---
- name: Añadir repositorio
  hosts: all
  become: yes
  tasks:
    - name: "Añadir repositorio OpenStack"
      apt_repository:
        repo: "deb http://ubuntu-cloud.archive.canonical.com/ubuntu xenial-updates/ocata main"
        state: present
```

**Eliminar repositorio:**
```yaml
tasks:
  - name: "Eliminar repositorio OpenStack"
    apt_repository:
      repo: "deb http://ubuntu-cloud.archive.canonical.com/ubuntu xenial-updates/ocata main"
      state: absent
```

### 5. Instalación de paquetes

El módulo **apt** gestiona paquetes en Ubuntu/Debian.

**Forma moderna (lista directa):**
```yaml
---
- name: Instalar paquetes
  hosts: all
  become: yes
  tasks:
    - name: Instalar Apache y PHP
      apt:
        name: 
          - apache2
          - php
          - libapache2-mod-php
        state: present
        update_cache: yes
```

**Usando variables:**
```yaml
---
- name: Instalar paquetes
  hosts: all
  become: yes
  vars:
    paquetes:
      - apache2
      - php
      - mysql-server
  tasks:
    - name: Instalar paquetes
      apt:
        name: "{{ paquetes }}"
        state: present
```

**Eliminar paquetes:**
```yaml
tasks:
  - name: Eliminar paquete
    apt:
      name: phpmyadmin
      state: absent
```

### 6. Ejecución de comandos en máquinas administradas

El módulo **shell** o **command** ejecuta comandos en las máquinas remotas.

```yaml
---
- name: Ejecutar comandos
  hosts: all
  tasks:
    - name: Copiar archivo
      shell: 'cp sample.txt sample.bak'
      args:
        chdir: /home/ubuntu
    
    - name: Obtener fecha
      command: date
      register: fecha_actual
    
    - name: Mostrar resultado
      debug:
        msg: "Fecha: {{ fecha_actual.stdout }}"
```

### 7. Manejo de archivos y directorios

El módulo **file** permite gestionar archivos y directorios.

```yaml
---
- name: Gestionar archivos
  hosts: all
  become: yes
  tasks:
    - name: Crear directorio
      file:
        path: /opt/mi_aplicacion
        state: directory
        owner: ubuntu
        group: ubuntu
        mode: '0755'
    
    - name: Crear archivo vacío
      file:
        path: /tmp/archivo.txt
        state: touch
        mode: '0644'
    
    - name: Eliminar archivo
      file:
        path: /tmp/archivo_viejo.txt
        state: absent
    
    - name: Crear enlace simbólico
      file:
        src: /opt/app/current
        dest: /opt/app/latest
        state: link
```

### 8. Copiar archivos

El módulo **copy** copia archivos desde el controlador a los nodos administrados.

```yaml
---
- name: Copiar archivos
  hosts: all
  tasks:
    - name: Copiar archivo de configuración
      copy:
        src: /home/ansible/config.conf
        dest: /etc/app/config.conf
        owner: root
        group: root
        mode: '0644'
        backup: yes    # Crear backup antes de sobrescribir
    
    - name: Crear archivo con contenido
      copy:
        content: "Contenido del archivo\nLínea 2\n"
        dest: /tmp/mi_archivo.txt
```

### 9. Descargar archivos desde internet

El módulo **get_url** descarga archivos desde URLs.

```yaml
---
- name: Descargar archivos
  hosts: all
  tasks:
    - name: Descargar script SQL
      get_url:
        url: https://raw.githubusercontent.com/usuario/repo/master/init.sql
        dest: /home/ubuntu/init.sql
        mode: '0644'
```

### 10. Reiniciar servicios

El módulo **service** administra servicios en nodos remotos.

```yaml
---
- name: Gestionar servicios
  hosts: all
  become: yes
  tasks:
    - name: Reiniciar Apache
      service:
        name: apache2
        state: restarted
    
    - name: Asegurar que MySQL está en ejecución
      service:
        name: mysql
        state: started
        enabled: yes    # Iniciar automáticamente al arrancar
    
    - name: Detener servicio
      service:
        name: cups
        state: stopped
        enabled: no
```

**Reiniciar múltiples servicios:**
```yaml
tasks:
  - name: Reiniciar servicios
    service:
      name: "{{ item }}"
      state: restarted
    loop:
      - apache2
      - mysql
      - redis
```

### 11. Reiniciar servidores y esperar

```yaml
---
- name: Reiniciar y esperar
  hosts: all
  become: yes
  tasks:
    - name: Reiniciar servidor
      shell: sleep 2 && reboot
      async: 1
      poll: 0
      ignore_errors: yes
    
    - name: Esperar a que el servidor vuelva
      wait_for_connection:
        delay: 15        # Esperar 15 segundos antes de empezar a comprobar
        sleep: 10        # Comprobar cada 10 segundos
        timeout: 300     # Timeout de 5 minutos
    
    - name: Confirmar que está activo
      debug:
        msg: "{{ inventory_hostname }} está activo"
```

## Buenas prácticas para Playbooks

1. **Nombres descriptivos:** Usa nombres claros y descriptivos para plays y tareas
2. **Comentarios:** Añade comentarios para explicar lógica compleja
3. **Un objetivo por playbook:** Cada playbook debe tener un propósito claro
4. **Idempotencia:** Asegura que las tareas sean idempotentes
5. **Manejo de errores:** Usa `ignore_errors`, `failed_when`, `changed_when` cuando sea necesario
6. **Variables externas:** Externaliza configuraciones en archivos de variables
7. **Testing:** Prueba con `--check` y `--diff` antes de aplicar en producción
8. **Roles para complejidad:** Para playbooks complejos, organiza en roles
9. **Control de versiones:** Guarda playbooks en Git
10. **Documentación:** Incluye README explicando el propósito del playbook

## Resumen

- Los **playbooks** son la unidad principal de automatización en Ansible
- Se escriben en **YAML** de forma declarativa e idempotente
- Componentes principales: **hosts, tasks, vars, handlers, roles**
- Se ejecutan con `ansible-playbook nombre.yml`
- Permiten **múltiples plays** en un mismo archivo
- **Gathering facts** recopila información automáticamente de los hosts
- Múltiples **módulos** disponibles para diferentes operaciones
- Ansible es **idempotente**: ejecutar múltiples veces es seguro
- Las **buenas prácticas** garantizan código mantenible y robusto

---
    - name: Instalar Apache
      apt:
        name: "{{ paquete }}"
        state: present
      notify: Reiniciar Apache
    - name: Copiar página de inicio
      copy:
        content: "¡Bienvenido a mi web!"
        dest: "{{ archivo_index }}"
  handlers:
    - name: Reiniciar Apache
      service:
        name: apache2
        state: restarted
```

## Includes y reutilización
Se pueden incluir tareas, handlers o variables desde otros archivos usando `include_tasks`, `import_tasks` o `vars_files`.

Los playbooks son la base para la automatización avanzada y la gestión de infraestructuras como código.