# 5. Módulos de Ansible

## ¿Qué son los Módulos?

Los **módulos** son el núcleo funcional de Ansible. Cada tarea de un playbook utiliza un módulo para realizar una acción concreta sobre los hosts gestionados. Son **componentes reutilizables** que encapsulan funcionalidad específica.

Los módulos se ejecutan en los hosts administrados y realizan operaciones como:
- Instalar paquetes
- Copiar archivos
- Gestionar servicios
- Crear usuarios
- Configurar redes
- Y mucho más...

## Características de los Módulos

- **Idempotentes:** La mayoría son idempotentes (se pueden ejecutar múltiples veces sin efectos secundarios)
- **Multiplataforma:** Muchos módulos funcionan en diferentes sistemas operativos
- **Extensibles:** Puedes crear módulos personalizados
- **Bien documentados:** Cada módulo tiene documentación completa
- **Especializados:** Cada módulo tiene un propósito específico

## Tipos de módulos

### 1. Core Modules (Módulos principales)
Incluidos por defecto en Ansible y mantenidos por el equipo core:
- `file`, `copy`, `template`
- `apt`, `yum`, `package`
- `service`, `systemd`
- `user`, `group`
- `shell`, `command`

### 2. Collections
Módulos organizados en colecciones temáticas:
- `ansible.builtin.*` - Módulos básicos
- `community.general.*` - Módulos de la comunidad
- `ansible.posix.*` - Módulos POSIX
- Cloud providers (AWS, Azure, GCP, etc.)

### 3. Módulos personalizados
Puedes crear tus propios módulos en Python o cualquier lenguaje compatible.

## Documentación de módulos

### Consultar documentación desde la línea de comandos

```bash
# Ver documentación de un módulo
ansible-doc apt

# Listar todos los módulos disponibles
ansible-doc -l

# Buscar módulos relacionados con servicios
ansible-doc -l | grep service

# Ver ejemplos de uso de un módulo
ansible-doc apt -s
```

### Documentación online
- [Índice de módulos oficial](https://docs.ansible.com/ansible/latest/collections/index.html)
- [Módulos por categoría](https://docs.ansible.com/ansible/latest/modules/modules_by_category.html)

## Categorías principales de módulos

### 1. Gestión de paquetes
- `apt` - Gestión de paquetes en Debian/Ubuntu
- `yum` / `dnf` - Gestión de paquetes en RedHat/CentOS/Fedora
- `package` - Gestión genérica de paquetes (detecta el sistema)
- `pip` - Gestión de paquetes Python
- `npm` - Gestión de paquetes Node.js
- `gem` - Gestión de paquetes Ruby

### 2. Gestión de archivos
- `file` - Crear, eliminar, modificar archivos y directorios
- `copy` - Copiar archivos desde el controlador
- `template` - Procesar plantillas Jinja2
- `lineinfile` - Asegurar línea específica en archivo
- `blockinfile` - Insertar/actualizar/eliminar bloque de texto
- `fetch` - Descargar archivos desde nodos remotos
- `get_url` - Descargar desde URLs
- `unarchive` - Descomprimir archivos

### 3. Gestión de servicios
- `service` - Gestionar servicios del sistema
- `systemd` - Control avanzado de servicios systemd

### 4. Usuarios y grupos
- `user` - Gestionar cuentas de usuario
- `group` - Gestionar grupos
- `authorized_key` - Gestionar claves SSH autorizadas

### 5. Ejecución de comandos
- `command` - Ejecutar comandos (sin shell)
- `shell` - Ejecutar comandos con shell
- `script` - Ejecutar scripts locales en hosts remotos
- `raw` - Ejecutar comandos sin Python (útil para inicialización)

### 6. Control de flujo
- `debug` - Imprimir mensajes y variables
- `fail` - Forzar fallo del playbook
- `assert` - Verificar condiciones
- `wait_for` - Esperar condición específica
- `pause` - Pausar ejecución

### 7. Bases de datos
- `mysql_db`, `mysql_user` - MySQL/MariaDB
- `postgresql_db`, `postgresql_user` - PostgreSQL
- `mongodb_user` - MongoDB

### 8. Cloud
- `amazon.aws.*` - AWS (EC2, S3, RDS, etc.)
- `azure.azcollection.*` - Microsoft Azure
- `google.cloud.*` - Google Cloud Platform
- `openstack.*` - OpenStack

### 9. Redes
- `uri` - Interactuar con APIs HTTP/HTTPS
- `get_url` - Descargar desde URLs
- `firewalld` - Gestionar firewall
- `iptables` - Gestionar reglas iptables

### 10. Docker y contenedores
- `docker_container` - Gestionar contenedores Docker
- `docker_image` - Gestionar imágenes Docker
- `docker_network` - Gestionar redes Docker

## Módulos más utilizados con ejemplos detallados

### 1. Módulo `apt` - Gestión de paquetes (Debian/Ubuntu)

**Instalar paquetes:**
```yaml
- name: Instalar Apache
  apt:
    name: apache2
    state: present          # present, absent, latest
    update_cache: yes       # Actualizar cache antes
    cache_valid_time: 3600  # Cache válido por 1 hora
```

**Instalar múltiples paquetes:**
```yaml
- name: Instalar varios paquetes
  apt:
    name:
      - apache2
      - php
      - mysql-server
    state: latest
    update_cache: yes
```

**Actualizar todos los paquetes:**
```yaml
- name: Actualizar sistema
  apt:
    upgrade: dist    # dist, full, yes, safe
    update_cache: yes
```

**Eliminar paquetes:**
```yaml
- name: Eliminar paquete
  apt:
    name: apache2
    state: absent
    purge: yes    # Eliminar también archivos de configuración
```

### 2. Módulo `copy` - Copiar archivos

**Copiar archivo:**
```yaml
- name: Copiar archivo de configuración
  copy:
    src: /ansible/files/config.conf    # Origen (en controlador)
    dest: /etc/app/config.conf          # Destino (en nodo remoto)
    owner: root
    group: root
    mode: '0644'
    backup: yes    # Crear backup antes de sobrescribir
```

**Crear archivo con contenido:**
```yaml
- name: Crear archivo con contenido
  copy:
    content: |
      # Configuración de la aplicación
      APP_NAME=MiApp
      APP_ENV=production
    dest: /etc/app/.env
    mode: '0600'
```

### 3. Módulo `file` - Gestionar archivos y directorios

**Crear directorio:**
```yaml
- name: Crear directorio
  file:
    path: /opt/aplicacion
    state: directory
    owner: ubuntu
    group: ubuntu
    mode: '0755'
    recurse: yes    # Aplicar permisos recursivamente
```

**Crear archivo vacío:**
```yaml
- name: Crear archivo vacío
  file:
    path: /tmp/archivo.txt
    state: touch
    mode: '0644'
```

**Eliminar archivo o directorio:**
```yaml
- name: Eliminar directorio
  file:
    path: /tmp/directorio_viejo
    state: absent
```

**Crear enlace simbólico:**
```yaml
- name: Crear enlace simbólico
  file:
    src: /opt/app/releases/v1.2.0
    dest: /opt/app/current
    state: link
```

**Cambiar permisos:**
```yaml
- name: Cambiar permisos
  file:
    path: /var/www/html
    owner: www-data
    group: www-data
    mode: '0755'
    recurse: yes
```

### 4. Módulo `template` - Procesar plantillas Jinja2

**Archivo de plantilla (templates/httpd.conf.j2):**
```jinja2
ServerName {{ server_name }}
Listen {{ http_port }}
DocumentRoot {{ document_root }}

<VirtualHost *:{{ http_port }}>
    ServerAdmin {{ admin_email }}
    ServerName {{ server_name }}
</VirtualHost>
```

**Usar la plantilla:**
```yaml
- name: Desplegar configuración de Apache
  template:
    src: templates/httpd.conf.j2
    dest: /etc/apache2/httpd.conf
    owner: root
    group: root
    mode: '0644'
    validate: 'apache2ctl -t -f %s'    # Validar antes de aplicar
  notify: Reiniciar Apache
```

### 5. Módulo `service` - Gestionar servicios

**Iniciar servicio:**
```yaml
- name: Iniciar Apache
  service:
    name: apache2
    state: started       # started, stopped, restarted, reloaded
    enabled: yes         # Habilitar al arranque
```

**Reiniciar servicio:**
```yaml
- name: Reiniciar servicio
  service:
    name: apache2
    state: restarted
```

**Detener y deshabilitar:**
```yaml
- name: Detener y deshabilitar servicio
  service:
    name: cups
    state: stopped
    enabled: no
```

### 6. Módulo `user` - Gestionar usuarios

**Crear usuario:**
```yaml
- name: Crear usuario deploy
  user:
    name: deploy
    comment: "Usuario de despliegue"
    shell: /bin/bash
    home: /home/deploy
    createhome: yes
    groups: sudo,www-data
    append: yes         # Añadir a grupos, no reemplazar
    password: "{{ 'password' | password_hash('sha512') }}"
    state: present
```

**Eliminar usuario:**
```yaml
- name: Eliminar usuario
  user:
    name: usuario_viejo
    state: absent
    remove: yes    # Eliminar también directorio home
```

### 7. Módulo `group` - Gestionar grupos

```yaml
- name: Crear grupo
  group:
    name: developers
    gid: 1500
    state: present
```

### 8. Módulo `lineinfile` - Modificar líneas en archivos

**Asegurar que existe una línea:**
```yaml
- name: Añadir línea a archivo
  lineinfile:
    path: /etc/hosts
    line: "192.168.1.10 server1.example.com"
    state: present
```

**Reemplazar línea usando regex:**
```yaml
- name: Cambiar puerto SSH
  lineinfile:
    path: /etc/ssh/sshd_config
    regexp: '^#?Port'
    line: 'Port 2222'
  notify: Reiniciar SSH
```

**Insertar después de una línea:**
```yaml
- name: Añadir configuración
  lineinfile:
    path: /etc/config.conf
    insertafter: '^\[section\]'
    line: 'new_option=value'
```

### 9. Módulo `shell` / `command` - Ejecutar comandos

**Usando command (sin shell):**
```yaml
- name: Obtener fecha
  command: date +%Y-%m-%d
  register: fecha_actual
  changed_when: false    # No marcar como cambio

- name: Mostrar fecha
  debug:
    var: fecha_actual.stdout
```

**Usando shell (con shell):**
```yaml
- name: Ejecutar script complejo
  shell: |
    cd /opt/app
    source venv/bin/activate
    python manage.py migrate
  args:
    executable: /bin/bash
    chdir: /opt/app
```

**Ejecutar solo si condición:**
```yaml
- name: Crear backup si no existe
  command: cp database.db database.db.bak
  args:
    chdir: /var/lib/app
    creates: /var/lib/app/database.db.bak    # Solo si no existe
```

### 10. Módulo `get_url` - Descargar desde URLs

```yaml
- name: Descargar archivo
  get_url:
    url: https://example.com/archivo.tar.gz
    dest: /tmp/archivo.tar.gz
    mode: '0644'
    checksum: sha256:abc123...    # Verificar checksum
    timeout: 30
```

### 11. Módulo `debug` - Depuración

**Mostrar mensaje:**
```yaml
- name: Mostrar mensaje
  debug:
    msg: "El servidor es {{ inventory_hostname }}"
```

**Mostrar variable:**
```yaml
- name: Mostrar todas las facts
  debug:
    var: ansible_facts

- name: Mostrar variable específica
  debug:
    var: mi_variable
```

**Debug con verbosidad:**
```yaml
- name: Debug con verbosidad 2
  debug:
    msg: "Este mensaje solo aparece con -vv"
    verbosity: 2
```

### 12. Módulo `wait_for` - Esperar condición

**Esperar a que puerto esté disponible:**
```yaml
- name: Esperar a que Apache esté listo
  wait_for:
    port: 80
    delay: 5        # Esperar 5 segundos antes de empezar
    timeout: 300    # Timeout de 5 minutos
```

**Esperar a que archivo exista:**
```yaml
- name: Esperar archivo de lock
  wait_for:
    path: /var/lock/app.lock
    state: present
```

### 13. Módulo `uri` - Interactuar con APIs HTTP

```yaml
- name: Verificar que la aplicación responde
  uri:
    url: http://localhost:8080/health
    method: GET
    status_code: 200
    return_content: yes
  register: health_check

- name: Hacer POST a API
  uri:
    url: https://api.example.com/endpoint
    method: POST
    body_format: json
    body:
      key: value
    headers:
      Authorization: "Bearer {{ api_token }}"
    status_code: 201
```

## Uso de módulos con bucles

**Instalar múltiples paquetes:**
```yaml
- name: Instalar paquetes
  apt:
    name: "{{ item }}"
    state: present
  loop:
    - vim
    - git
    - curl
```

**Crear múltiples usuarios:**
```yaml
- name: Crear usuarios
  user:
    name: "{{ item.name }}"
    groups: "{{ item.groups }}"
    state: present
  loop:
    - { name: 'user1', groups: 'developers' }
    - { name: 'user2', groups: 'admins' }
    - { name: 'user3', groups: 'developers' }
```

## Registro de resultados con `register`

Captura la salida de un módulo para usar en tareas posteriores:

```yaml
- name: Obtener contenido de archivo
  command: cat /etc/hostname
  register: hostname_content

- name: Mostrar contenido
  debug:
    msg: "El hostname es: {{ hostname_content.stdout }}"

- name: Usar resultado en condicional
  debug:
    msg: "Es el servidor de producción"
  when: "'production' in hostname_content.stdout"
```

## Control de cambios

### `changed_when` - Definir cuándo marcar como cambio

```yaml
- name: Verificar si servicio está activo
  command: systemctl is-active apache2
  register: result
  changed_when: false    # Nunca marcar como cambio
  failed_when: result.rc not in [0, 3]
```

### `failed_when` - Definir cuándo marcar como fallo

```yaml
- name: Ejecutar script
  shell: /opt/script.sh
  register: script_result
  failed_when: "'ERROR' in script_result.stdout"
```

### `ignore_errors` - Continuar a pesar de errores

```yaml
- name: Intentar detener servicio
  service:
    name: servicio_opcional
    state: stopped
  ignore_errors: yes
```

## Módulos especializados

### Módulos MySQL/MariaDB

```yaml
# Instalar dependencias primero
- name: Instalar PyMySQL
  pip:
    name: pymysql
    state: present

- name: Crear base de datos
  mysql_db:
    name: mi_aplicacion
    state: present
    login_user: root
    login_password: "{{ mysql_root_password }}"

- name: Crear usuario de base de datos
  mysql_user:
    name: app_user
    password: "{{ app_db_password }}"
    priv: 'mi_aplicacion.*:ALL'
    state: present
    login_user: root
    login_password: "{{ mysql_root_password }}"

- name: Importar base de datos
  mysql_db:
    name: mi_aplicacion
    state: import
    target: /tmp/database.sql
    login_user: root
    login_password: "{{ mysql_root_password }}"
```

### Módulos Git

```yaml
- name: Clonar repositorio
  git:
    repo: 'https://github.com/usuario/repositorio.git'
    dest: /opt/aplicacion
    version: main
    force: yes
    
- name: Actualizar repositorio
  git:
    repo: 'https://github.com/usuario/repositorio.git'
    dest: /opt/aplicacion
    version: v2.0.0
    update: yes
```

### Módulos de archivos comprimidos

```yaml
- name: Descomprimir archivo
  unarchive:
    src: /tmp/archivo.tar.gz
    dest: /opt/aplicacion
    remote_src: yes    # El archivo está en el nodo remoto

- name: Descomprimir desde controlador
  unarchive:
    src: files/app.tar.gz
    dest: /opt/aplicacion
    owner: ubuntu
    group: ubuntu
```

### Módulos de sistema

```yaml
- name: Configurar hostname
  hostname:
    name: server01.example.com

- name: Configurar timezone
  timezone:
    name: Europe/Madrid

- name: Añadir entrada a cron
  cron:
    name: "Backup diario"
    minute: "0"
    hour: "2"
    job: "/usr/local/bin/backup.sh"
    user: root
```

## Buenas prácticas para usar módulos

1. **Usar módulos específicos:** Preferir módulos específicos sobre `command` o `shell`
   ```yaml
   # ❌ Evitar
   - command: apt-get install apache2
   
   # ✅ Preferir
   - apt:
       name: apache2
       state: present
   ```

2. **Aprovechar la idempotencia:** Los módulos nativos son idempotentes
   ```yaml
   # Es seguro ejecutar múltiples veces
   - name: Asegurar que Apache está instalado
     apt:
       name: apache2
       state: present
   ```

3. **Usar `check_mode` para testing:**
   ```yaml
   - name: Instalar paquete
     apt:
       name: apache2
       state: present
     check_mode: yes    # Simular sin hacer cambios
   ```

4. **Documentar tareas complejas:**
   ```yaml
   - name: Configurar Apache (modifica puerto y SSL)
     template:
       src: apache.conf.j2
       dest: /etc/apache2/apache.conf
     # Nota: Este cambio requiere reinicio del servicio
     notify: Reiniciar Apache
   ```

5. **Validar antes de aplicar:**
   ```yaml
   - name: Actualizar configuración
     template:
       src: nginx.conf.j2
       dest: /etc/nginx/nginx.conf
       validate: 'nginx -t -c %s'    # Validar sintaxis
   ```

6. **Manejar errores apropiadamente:**
   ```yaml
   - name: Intentar operación opcional
     command: /opt/script_opcional.sh
     register: result
     failed_when: result.rc not in [0, 2]  # 0 y 2 son OK
     ignore_errors: no
   ```

7. **Registrar y usar resultados:**
   ```yaml
   - name: Verificar si archivo existe
     stat:
       path: /etc/app/config.conf
     register: config_file
   
   - name: Crear configuración si no existe
     template:
       src: config.conf.j2
       dest: /etc/app/config.conf
     when: not config_file.stat.exists
   ```

8. **Usar tags para organización:**
   ```yaml
   - name: Instalar paquetes
     apt:
       name: apache2
       state: present
     tags:
       - install
       - apache
       - webserver
   ```

9. **Preferir módulos nativos sobre scripts:**
   ```yaml
   # ❌ Menos recomendado
   - name: Crear usuario
     shell: useradd -m -s /bin/bash usuario
   
   # ✅ Recomendado
   - name: Crear usuario
     user:
       name: usuario
       shell: /bin/bash
       createhome: yes
   ```

10. **Usar `become` solo cuando sea necesario:**
    ```yaml
    - name: Operación que requiere root
      apt:
        name: apache2
        state: present
      become: yes    # Solo para esta tarea
    ```

## Módulos por casos de uso

### Despliegue de aplicación web completa

```yaml
---
- name: Desplegar aplicación web
  hosts: webservers
  become: yes
  
  tasks:
    - name: Instalar dependencias del sistema
      apt:
        name:
          - nginx
          - python3-pip
          - python3-venv
          - git
        state: present
        update_cache: yes
    
    - name: Crear usuario de aplicación
      user:
        name: appuser
        shell: /bin/bash
        createhome: yes
    
    - name: Clonar repositorio
      git:
        repo: 'https://github.com/usuario/app.git'
        dest: /opt/miapp
        version: main
      become_user: appuser
    
    - name: Instalar dependencias Python
      pip:
        requirements: /opt/miapp/requirements.txt
        virtualenv: /opt/miapp/venv
        virtualenv_python: python3
      become_user: appuser
    
    - name: Configurar Nginx
      template:
        src: nginx.conf.j2
        dest: /etc/nginx/sites-available/miapp
        validate: 'nginx -t -c /dev/stdin < %s'
      notify: Reiniciar Nginx
    
    - name: Habilitar sitio
      file:
        src: /etc/nginx/sites-available/miapp
        dest: /etc/nginx/sites-enabled/miapp
        state: link
      notify: Reiniciar Nginx
    
    - name: Asegurar que Nginx está en ejecución
      service:
        name: nginx
        state: started
        enabled: yes
  
  handlers:
    - name: Reiniciar Nginx
      service:
        name: nginx
        state: restarted
```

## Resumen

- Los **módulos** son la base de la funcionalidad de Ansible
- Existen módulos para casi cualquier tarea de administración
- Categorías principales: paquetes, archivos, servicios, usuarios, comandos, cloud, BD
- Los módulos nativos son **idempotentes** y seguros
- Usar `ansible-doc` para consultar documentación
- **Preferir módulos específicos** sobre comandos shell
- Aprovechar **register** para capturar y usar resultados
- Usar control de flujo: `when`, `changed_when`, `failed_when`, `ignore_errors`
- Los módulos hacen que Ansible sea **potente y versátil**

## Recursos adicionales

- [Índice completo de módulos](https://docs.ansible.com/ansible/latest/collections/index.html)
- [Módulos por categoría](https://docs.ansible.com/ansible/latest/modules/modules_by_category.html)
- [Ansible Galaxy - Collections](https://galaxy.ansible.com/)
- Comando: `ansible-doc nombre_modulo`

---