# Formato y Estructura de Playbooks en Ansible

Un **playbook** es un archivo YAML que describe una o varias tareas a ejecutar sobre uno o varios hosts. Es la unidad principal de automatización en Ansible y representa el corazón de la herramienta.

## ¿Qué es un Playbook?

Los playbooks son **recetas** que definen el estado deseado de los sistemas. Se escriben en sintaxis YAML, que es fácil de leer y escribir.

Los playbooks y los roles se basan en la sintaxis YAML. (Ver apéndice YAML para más detalles sobre la sintaxis)

## Estructura básica de un Playbook

Los playbooks siguen esta estructura:

1. **Nombre del Playbook** (opcional pero recomendado)
2. **Gather facts** - Obtener información de los hosts administrados
3. **Hosts** - Sobre qué hosts aplicar el playbook
4. **Lista de tareas** - Qué hacer en esos hosts

**Formato básico:**
```yaml
---                           # Inicio del documento YAML

- name: Nombre descriptivo del play
  hosts: grupo_o_host         # Hosts objetivo
  gather_facts: yes           # Recopilar información (por defecto: yes)
  become: yes                 # Ejecutar como root (opcional)
  vars:                       # Variables locales (opcional)
    variable1: valor1
    variable2: valor2
  
  tasks:                      # Lista de tareas a ejecutar
    - name: Descripción de la tarea
      módulo:
        argumento1: valor
        argumento2: valor
  
  handlers:                   # Tareas especiales (opcional)
    - name: Nombre del handler
      módulo:
        argumento: valor
  
  roles:                      # Roles a incluir (opcional)
    - nombre_rol
```

## Componentes principales

### 1. name
Descripción legible del play o tarea. Aparece en la salida cuando se ejecuta el playbook.

```yaml
- name: Configurar servidor web Apache
  hosts: webservers
  tasks:
    - name: Instalar Apache
      apt:
        name: apache2
        state: present
```

### 2. hosts
Define los hosts o grupos objetivo donde se ejecutarán las tareas.

**Opciones:**
- `all` - Todos los hosts del inventario
- `webservers` - Grupo específico
- `webservers:databases` - Múltiples grupos
- `all:!databases` - Todos excepto databases
- `web1.example.com` - Host específico

### 3. become
Escala privilegios (ejecutar como root usando sudo).

```yaml
- name: Instalar paquetes del sistema
  hosts: all
  become: yes      # Ejecutar como root
  tasks:
    - name: Actualizar cache de paquetes
      apt:
        update_cache: yes
```

### 4. gather_facts
Recoge información del sistema (por defecto en `yes`). Esta información incluye: hostname, IP, SO, memoria, CPU, etc.

```yaml
- name: Playbook sin recopilar información
  hosts: all
  gather_facts: false    # Desactivar para mayor velocidad
  tasks:
    - name: Mostrar mensaje
      debug:
        msg: "Hola Ansible"
```

**Nota:** Desactivar `gather_facts` hace la ejecución más rápida cuando no necesitas información de los hosts.

### 5. vars
Define variables locales al play.

```yaml
- name: Desplegar aplicación
  hosts: webservers
  vars:
    app_name: mi_aplicacion
    app_port: 8080
  tasks:
    - name: Crear directorio de aplicación
      file:
        path: "/opt/{{ app_name }}"
        state: directory
```

### 6. tasks
Lista de tareas a ejecutar secuencialmente.

```yaml
tasks:
  - name: Primera tarea
    módulo1:
      parámetro: valor
  
  - name: Segunda tarea
    módulo2:
      parámetro: valor
```

### 7. handlers
Tareas especiales que se ejecutan **solo cuando son notificadas** por otra tarea. Se ejecutan al final del play.

```yaml
tasks:
  - name: Modificar configuración de Apache
    copy:
      src: httpd.conf
      dest: /etc/apache2/httpd.conf
    notify: Reiniciar Apache    # Notifica al handler

handlers:
  - name: Reiniciar Apache
    service:
      name: apache2
      state: restarted
```

### 8. roles
Incluye roles para modularizar la configuración.

```yaml
- hosts: webservers
  roles:
    - common
    - apache
    - php
```

### 9. tags
Permite etiquetar tareas para ejecución selectiva.

```yaml
tasks:
  - name: Instalar Apache
    apt:
      name: apache2
      state: present
    tags:
      - install
      - apache
  
  - name: Configurar Apache
    template:
      src: httpd.conf.j2
      dest: /etc/apache2/httpd.conf
    tags:
      - configure
      - apache
```

**Ejecutar solo tareas con tags específicos:**
```bash
ansible-playbook playbook.yml --tags "install"
ansible-playbook playbook.yml --skip-tags "configure"
```

## Ejemplos completos

### Ejemplo 1: Playbook mínimo

```yaml
---
- name: Verificar conectividad
  hosts: all
  tasks:
    - name: Comprobar conexión
      ping:
```

**Ejecución:**
```bash
ansible-playbook ping.yml
```

### Ejemplo 2: Playbook con gather_facts y variables

```yaml
---
- name: Mostrar información del sistema
  hosts: localhost
  gather_facts: true
  tasks:
    - name: Hacer ping
      ping:

    - name: Mostrar información
      debug:
        msg: "Máquina: {{ ansible_hostname }}, OS: {{ ansible_distribution }}"
```

**Salida esperada:**
```
TASK [Mostrar información] 
ok: [localhost] => {
    "msg": "Máquina: ansible-control, OS: Ubuntu"
}
```

### Ejemplo 3: Playbook con variables y handlers

```yaml
---
- name: Desplegar página web
  hosts: webservers
  become: yes
  vars:
    mensaje: "¡Hola desde Ansible!"
    puerto: 80
  
  tasks:
    - name: Instalar Apache
      apt:
        name: apache2
        state: present
        update_cache: yes
    
    - name: Crear página de inicio
      copy:
        content: "<h1>{{ mensaje }}</h1>"
        dest: /var/www/html/index.html
      notify: Reiniciar Apache
    
    - name: Asegurar que Apache está en ejecución
      service:
        name: apache2
        state: started
        enabled: yes
  
  handlers:
    - name: Reiniciar Apache
      service:
        name: apache2
        state: restarted
```

### Ejemplo 4: Playbook con roles

```yaml
---
- name: Configurar servidor completo
  hosts: webservers
  become: yes
  roles:
    - common          # Configuración básica
    - security        # Configuración de seguridad
    - apache          # Instalación de Apache
```

### Ejemplo 5: Múltiples plays en un playbook

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
```

### Ejemplo 6: Playbook con condicionales

```yaml
---
- name: Instalar paquetes según el SO
  hosts: all
  become: yes
  tasks:
    - name: Instalar Apache en Debian/Ubuntu
      apt:
        name: apache2
        state: present
      when: ansible_os_family == "Debian"
    
    - name: Instalar Apache en RedHat/CentOS
      yum:
        name: httpd
        state: present
      when: ansible_os_family == "RedHat"
```

### Ejemplo 7: Playbook con bucles

```yaml
---
- name: Crear múltiples usuarios
  hosts: all
  become: yes
  tasks:
    - name: Crear usuarios
      user:
        name: "{{ item }}"
        state: present
      loop:
        - usuario1
        - usuario2
        - usuario3
    
    - name: Instalar paquetes
      apt:
        name: "{{ item }}"
        state: present
      loop:
        - vim
        - git
        - curl
        - wget
```

## Ejecución de Playbooks

### Comando básico

```bash
ansible-playbook nombre-playbook.yml
```

### Parámetros de interés para ejecución de playbooks

**`-i archivo_de_inventario`:** Usa un archivo de inventario específico
```bash
ansible-playbook -i production/hosts.cfg deploy.yml
```

**`--become`:** Ejecuta operaciones como root
```bash
ansible-playbook playbook.yml --become
```

**`--start-at-task="nombre_tarea"`:** Comienza desde una tarea específica
```bash
ansible-playbook mysql.yml --start-at-task "Actualizar cache de paquetes"
```

**`--step`:** Ejecuta el playbook paso a paso (pide confirmación para cada tarea)
```bash
ansible-playbook playbook.yml --step
```

**`--check`:** Modo dry-run (no hace cambios reales)
```bash
ansible-playbook playbook.yml --check
```

**`--diff`:** Muestra las diferencias en los archivos modificados
```bash
ansible-playbook playbook.yml --diff
```

**`--tags`:** Ejecuta solo tareas con tags específicos
```bash
ansible-playbook playbook.yml --tags "install,configure"
```

**`--skip-tags`:** Omite tareas con tags específicos
```bash
ansible-playbook playbook.yml --skip-tags "testing"
```

**`--limit`:** Limita la ejecución a hosts específicos
```bash
ansible-playbook playbook.yml --limit "web1.example.com"
```

**`--extra-vars` o `-e`:** Pasa variables desde la línea de comandos
```bash
ansible-playbook playbook.yml -e "app_version=2.0 environment=production"
```

**`--list-tasks`:** Lista todas las tareas sin ejecutar
```bash
ansible-playbook playbook.yml --list-tasks
```

**`--list-hosts`:** Lista los hosts afectados sin ejecutar
```bash
ansible-playbook playbook.yml --list-hosts
```

**`-v`, `-vv`, `-vvv`, `-vvvv`:** Aumenta el nivel de verbosidad
```bash
ansible-playbook playbook.yml -vvv
```

### Ejemplo completo de ejecución

```bash
# Ejecutar playbook con múltiples opciones
ansible-playbook mysql.yml \
  --become \
  --start-at-task "Actualizar cache de paquetes" \
  --step \
  --diff
```

## Salida de ejecución de un Playbook

Al ejecutar un playbook, Ansible muestra el progreso y resultados:

```
PLAY [Nombre del playbook] *********************************************

TASK [Gathering Facts] *************************************************
ok: [host1]
ok: [host2]

TASK [Nombre de la tarea 1] ********************************************
changed: [host1]
changed: [host2]

TASK [Nombre de la tarea 2] ********************************************
ok: [host1]
ok: [host2]

PLAY RECAP *************************************************************
host1                      : ok=3    changed=1    unreachable=0    failed=0
host2                      : ok=3    changed=1    unreachable=0    failed=0
```

**Estados de las tareas:**
- **ok:** La tarea se ejecutó correctamente (sin cambios)
- **changed:** La tarea realizó cambios en el sistema
- **failed:** La tarea falló
- **skipped:** La tarea fue omitida (por condicionales)
- **unreachable:** No se pudo conectar al host

## Idempotencia en Playbooks

La **idempotencia** es una característica fundamental de Ansible. Significa que ejecutar el mismo playbook múltiples veces produce el mismo resultado.

**Ejemplo:**
```yaml
- name: Asegurar que Apache está instalado
  hosts: webservers
  become: yes
  tasks:
    - name: Instalar Apache
      apt:
        name: apache2
        state: present
```

**Primera ejecución:**
```
TASK [Instalar Apache] *************************************************
changed: [web1]    # Se instaló Apache
```

**Segunda ejecución:**
```
TASK [Instalar Apache] *************************************************
ok: [web1]    # Apache ya estaba instalado, no hay cambios
```

### Ventajas de la idempotencia

1. **Seguridad:** Puedes ejecutar playbooks repetidamente sin miedo
2. **Convergencia:** El sistema siempre converge al estado deseado
3. **Recuperación:** Puedes re-ejecutar tras un fallo parcial
4. **Mantenimiento:** Aplicar la misma configuración periódicamente

## Buenas prácticas para Playbooks

1. **Nombres descriptivos:** Usa nombres claros para plays y tareas
2. **Un objetivo por playbook:** Cada playbook debe tener un propósito claro
3. **Usar roles:** Para playbooks complejos, organiza en roles
4. **Comentarios:** Añade comentarios cuando sea necesario
5. **Idempotencia:** Asegura que las tareas sean idempotentes
6. **Manejo de errores:** Usa `ignore_errors`, `failed_when`, `changed_when`
7. **Variables:** Externaliza configuraciones en variables
8. **Testing:** Prueba con `--check` y `--diff` antes de aplicar
9. **Control de versiones:** Guarda playbooks en Git
10. **Documentación:** Incluye un README explicando el propósito

## Sintaxis YAML básica para Playbooks

### Inicio de documento
```yaml
---
# Tres guiones inician el documento YAML
```

### Listas
```yaml
# Formato de lista
tasks:
  - name: Tarea 1
    ping:
  - name: Tarea 2
    debug:
      msg: "Hola"
```

### Diccionarios/Mapas
```yaml
# Formato de diccionario
vars:
  app_name: mi_app
  app_port: 8080
  app_user: deploy
```

### Multilínea con `>`
```yaml
# El > convierte múltiples líneas en una sola
description: >
  Esta es una descripción larga
  que se convertirá en una sola línea
```

### Multilínea con `|`
```yaml
# El | preserva los saltos de línea
script: |
  #!/bin/bash
  echo "Línea 1"
  echo "Línea 2"
```

## Debugging de Playbooks

### Módulo debug

```yaml
- name: Mostrar información de depuración
  debug:
    msg: "El valor de la variable es: {{ mi_variable }}"

- name: Mostrar todas las variables
  debug:
    var: ansible_facts

- name: Debug condicional
  debug:
    msg: "Esto es producción"
  when: environment == "production"
```

### Verificar sintaxis

```bash
# Verificar sintaxis YAML
ansible-playbook playbook.yml --syntax-check
```

### Modo dry-run

```bash
# Ver qué cambios se harían sin aplicarlos
ansible-playbook playbook.yml --check --diff
```

## Resumen

- Los **playbooks** son archivos YAML que definen automatizaciones
- Componentes clave: **name, hosts, tasks, handlers, vars, roles**
- Se ejecutan con `ansible-playbook`
- Son **idempotentes**: ejecutar múltiples veces es seguro
- Permiten **condicionales, bucles, variables y handlers**
- Múltiples **parámetros** para controlar la ejecución
- **Sintaxis YAML** simple y legible
- Base fundamental para la automatización con Ansible

## Más información

- [Documentación oficial de Playbooks](https://docs.ansible.com/ansible/latest/user_guide/playbooks.html)
- [Best Practices de Ansible](https://docs.ansible.com/ansible/latest/user_guide/playbooks_best_practices.html)
- [Referencia YAML](https://yaml.org/spec/1.2/spec.html)

---
