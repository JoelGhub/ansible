# 7. Roles en Ansible

## ¿Qué son los Roles?

Los **roles** en Ansible son una forma avanzada de organizar y reutilizar configuraciones. Permiten agrupar tareas, variables, archivos, plantillas y handlers relacionados con una función o servicio específico, facilitando la **modularidad**, **reutilización** y **mantenimiento**.

En lugar de tener todas las tareas directamente en los playbooks, se organizan en roles que pueden ser reutilizados en diferentes proyectos.

## ¿Por qué usar Roles?

### Ventajas de los roles

1. **Reutilización:** Un rol se puede usar en múltiples playbooks y proyectos
2. **Organización:** Estructura clara y predecible
3. **Modularidad:** Cada rol tiene una responsabilidad específica
4. **Mantenibilidad:** Más fácil de mantener y actualizar
5. **Colaboración:** Se pueden compartir en Ansible Galaxy
6. **Testing:** Cada rol se puede probar independientemente
7. **Escalabilidad:** Facilita el crecimiento de proyectos complejos
8. **Documentación:** Estructura que facilita la documentación

### Cuándo usar Roles

- Cuando las tareas se vuelven complejas o numerosas
- Cuando quieres reutilizar configuraciones en varios proyectos
- Cuando trabajas en equipo y necesitas modularidad
- Cuando quieres compartir configuraciones con la comunidad

## Estructura de un Rol

Un rol tiene una estructura de carpetas predefinida:

```
nombre_rol/
├── tasks/           # Tareas principales del rol
│   └── main.yml
├── handlers/        # Handlers que pueden ser notificados
│   └── main.yml
├── templates/       # Plantillas Jinja2 (.j2)
│   └── config.j2
├── files/           # Archivos estáticos para copiar
│   └── script.sh
├── vars/            # Variables del rol (alta prioridad)
│   └── main.yml
├── defaults/        # Variables por defecto (baja prioridad)
│   └── main.yml
├── meta/            # Metadatos del rol (dependencias, autor, etc.)
│   └── main.yml
├── tests/           # Tests del rol
│   ├── inventory
│   └── test.yml
└── README.md        # Documentación del rol
```

### Descripción de cada directorio

#### `tasks/`
Contiene el archivo `main.yml` con la **lista de tareas** a ejecutar. Es el corazón del rol.

**Ejemplo `tasks/main.yml`:**
```yaml
---
# tasks file for apache

- name: Instalar Apache
  apt:
    name: apache2
    state: latest
    update_cache: yes

- name: Copiar configuración personalizada
  template:
    src: apache.conf.j2
    dest: /etc/apache2/apache2.conf
    owner: root
    group: root
    mode: '0644'
  notify: Reiniciar Apache

- name: Asegurar que Apache está en ejecución
  service:
    name: apache2
    state: started
    enabled: yes
```

#### `handlers/`
Contiene el archivo `main.yml` con **handlers** que pueden ser notificados por las tareas.

**Ejemplo `handlers/main.yml`:**
```yaml
---
# handlers file for apache

- name: Reiniciar Apache
  service:
    name: apache2
    state: restarted

- name: Recargar Apache
  service:
    name: apache2
    state: reloaded
```

#### `templates/`
Archivos de **plantillas Jinja2** (.j2) que se procesan y despliegan en los nodos administrados.

**Ejemplo `templates/apache.conf.j2`:**
```jinja2
ServerRoot "/etc/apache2"
Listen {{ apache_port }}
ServerName {{ server_name }}

MaxClients {{ max_clients }}
KeepAlive {{ keep_alive }}

<VirtualHost *:{{ apache_port }}>
    ServerAdmin {{ admin_email }}
    DocumentRoot {{ document_root }}
</VirtualHost>
```

#### `files/`
Archivos **estáticos** que se copian tal cual sin procesar (a diferencia de templates).

**Ejemplo:** Scripts, certificados, archivos de configuración estáticos, etc.

#### `vars/`
Variables específicas del rol con **alta prioridad** (sobrescriben defaults).

**Ejemplo `vars/main.yml`:**
```yaml
---
# vars file for apache
apache_package: apache2
apache_service: apache2
apache_config_dir: /etc/apache2
```

#### `defaults/`
Variables por defecto del rol con **baja prioridad** (pueden ser sobrescritas fácilmente).

**Ejemplo `defaults/main.yml`:**
```yaml
---
# defaults file for apache
apache_port: 80
max_clients: 200
keep_alive: On
server_name: localhost
admin_email: webmaster@localhost
document_root: /var/www/html
```

#### `meta/`
Metadatos del rol: autor, licencia, plataformas soportadas, **dependencias** de otros roles.

**Ejemplo `meta/main.yml`:**
```yaml
---
galaxy_info:
  author: Tu Nombre
  description: Instalación y configuración de Apache
  license: MIT
  min_ansible_version: 2.9
  
  platforms:
    - name: Ubuntu
      versions:
        - focal
        - jammy
    - name: Debian
      versions:
        - buster
        - bullseye
  
  galaxy_tags:
    - web
    - apache
    - httpd

dependencies: []
  # - role: common
  # - role: firewall
```

#### `tests/`
Casos de prueba para testing del rol.

**Ejemplo `tests/test.yml`:**
```yaml
---
- hosts: localhost
  remote_user: root
  roles:
    - apache
```

## Uso de Roles en Playbooks

### Forma básica

```yaml
---
- name: Configurar servidores web
  hosts: webservers
  become: yes
  roles:
    - common
    - apache
    - php
```

### Con variables

```yaml
---
- name: Configurar Apache con variables
  hosts: webservers
  become: yes
  roles:
    - role: apache
      apache_port: 8080
      max_clients: 500
      server_name: www.example.com
```

### Forma de diccionario

```yaml
---
- name: Configurar servicios
  hosts: webservers
  become: yes
  roles:
    - { role: apache, apache_port: 80 }
    - { role: php, php_version: 7.4 }
    - { role: mysql, mysql_root_password: "secret" }
```

### Con tags

```yaml
---
- name: Configurar servidor completo
  hosts: webservers
  become: yes
  roles:
    - { role: common, tags: ['common', 'base'] }
    - { role: apache, tags: ['apache', 'web'] }
    - { role: mysql, tags: ['mysql', 'database'] }
```

**Ejecutar solo roles específicos:**
```bash
ansible-playbook site.yml --tags "web"
```

### Con condicionales

```yaml
---
- name: Configurar según el entorno
  hosts: all
  become: yes
  roles:
    - role: apache
      when: ansible_os_family == "Debian"
    
    - role: httpd
      when: ansible_os_family == "RedHat"
```

## Organización de Proyectos con Roles

### Estructura recomendada

En un modo de funcionamiento normal, las tareas no están directamente en los playbooks, sino organizadas en roles.

```
proyecto/
├── ansible.cfg              # Configuración de Ansible
├── hosts.cfg                # Inventario de hosts
├── group_vars/              # Variables por grupo
│   ├── all.yml
│   ├── webservers.yml
│   └── databases.yml
├── host_vars/               # Variables por host
│   ├── web1.yml
│   └── db1.yml
├── roles/                   # Directorio de roles
│   ├── common/
│   │   ├── tasks/
│   │   │   └── main.yml
│   │   ├── handlers/
│   │   │   └── main.yml
│   │   └── templates/
│   ├── apache/
│   │   ├── tasks/
│   │   ├── handlers/
│   │   ├── templates/
│   │   └── defaults/
│   ├── mysql/
│   │   ├── tasks/
│   │   ├── handlers/
│   │   └── vars/
│   └── php/
│       ├── tasks/
│       └── defaults/
├── site.yml                 # Playbook principal
├── webservers.yml           # Playbook para webservers
└── databases.yml            # Playbook para databases
```

### Playbook principal (site.yml)

Para proyectos grandes, se puede crear un playbook maestro que incluye otros:

**Ejemplo `site.yml`:**
```yaml
---
# Playbook principal que ejecuta todos los demás
- import_playbook: common.yml
- import_playbook: webservers.yml
- import_playbook: databases.yml
```

### Playbook con roles (webservers.yml)

```yaml
---
- name: Configurar servidores web
  hosts: webservers
  become: yes
  roles:
    - common
    - apache
    - php

- name: Configurar servidores de aplicación
  hosts: appservers
  become: yes
  roles:
    - common
    - nodejs
    - nginx
```

### Ejemplo completo de proyecto

```yaml
---
# Todos los servidores reciben configuración común
- hosts: all
  become: true
  roles:
    - basic

# Servidor controlador
- hosts: controller
  become: true
  roles:
    - ntp_server
    - sql_database
    - rabbitmq
    - memcached

# Servidores de red
- hosts: network
  become: true
  roles:
    - ntp_others
    - network_config

# Servidores de cómputo
- hosts: compute
  become: true
  roles:
    - ntp_others
    - compute_config

# Todos reciben paquetes de OpenStack
- hosts: all
  become: true
  roles:
    - openstack_packages
```

## Creación de Roles

### Método 1: Crear manualmente

Puedes crear la estructura de directorios manualmente:

```bash
mkdir -p roles/apache/{tasks,handlers,templates,files,vars,defaults,meta}
touch roles/apache/tasks/main.yml
touch roles/apache/handlers/main.yml
touch roles/apache/defaults/main.yml
```

### Método 2: Usar `ansible-galaxy init`

**Ansible Galaxy** proporciona un comando para inicializar la estructura:

```bash
# Crear rol en el directorio actual
ansible-galaxy init apache

# Crear rol en directorio específico
ansible-galaxy init roles/apache

# Ver la estructura creada
tree roles/apache
```

**Salida:**
```
roles/apache/
├── defaults
│   └── main.yml
├── files
├── handlers
│   └── main.yml
├── meta
│   └── main.yml
├── README.md
├── tasks
│   └── main.yml
├── templates
├── tests
│   ├── inventory
│   └── test.yml
└── vars
    └── main.yml
```

### Convención de nombres

Seguir la convención: `usuario.nombre_rol`

```bash
ansible-galaxy init ualmtorres.apache
ansible-galaxy init geerlingguy.mysql
```

Esto permite tener varios roles similares de diferentes autores.

## Ansible Galaxy

### ¿Qué es Ansible Galaxy?

**Ansible Galaxy** es el repositorio oficial de roles y colecciones de Ansible, donde la comunidad comparte roles reutilizables para instalar, configurar y gestionar servicios y aplicaciones.

- **URL:** https://galaxy.ansible.com
- Miles de roles disponibles
- Roles mantenidos por la comunidad y autores oficiales
- Búsqueda y calificaciones de roles

### Comandos de Ansible Galaxy

#### Buscar roles

```bash
# Buscar roles relacionados con Apache
ansible-galaxy search apache

# Buscar con filtros
ansible-galaxy search apache --author geerlingguy
```

#### Instalar roles

```bash
# Instalar un rol en el directorio por defecto (~/.ansible/roles/)
ansible-galaxy install geerlingguy.apache

# Instalar en un directorio específico
ansible-galaxy install geerlingguy.apache -p roles/

# Instalar versión específica
ansible-galaxy install geerlingguy.apache,2.0.0

# Instalar desde archivo requirements.yml
ansible-galaxy install -r requirements.yml
```

#### Listar roles instalados

```bash
# Listar roles instalados
ansible-galaxy list

# Listar roles en directorio específico
ansible-galaxy list -p roles/
```

#### Eliminar roles

```bash
# Eliminar un rol
ansible-galaxy remove geerlingguy.apache
```

#### Ver información de un rol

```bash
# Ver información detallada
ansible-galaxy info geerlingguy.apache
```

### Archivo requirements.yml

Para proyectos con múltiples roles, usa un archivo `requirements.yml`:

**Ejemplo `requirements.yml`:**
```yaml
---
# Desde Ansible Galaxy
- name: geerlingguy.apache
  version: 3.1.4

- name: geerlingguy.php
  version: 4.8.0

- name: geerlingguy.mysql
  version: 3.3.2

# Desde repositorio Git
- src: https://github.com/usuario/mi-rol.git
  name: mi-rol
  version: main

# Desde URL de archivo tar.gz
- src: https://example.com/roles/mi-rol.tar.gz
  name: mi-rol
```

**Instalar todos los roles:**
```bash
ansible-galaxy install -r requirements.yml -p roles/
```

### Ejemplo completo: Usar roles de Galaxy

#### 1. Instalar roles necesarios

```bash
# Crear directorio de roles
mkdir -p roles

# Instalar roles
ansible-galaxy install geerlingguy.apache -p roles/
ansible-galaxy install geerlingguy.php -p roles/
ansible-galaxy install geerlingguy.mysql -p roles/
```

#### 2. Crear playbook

**`webserver.yml`:**
```yaml
---
- name: Configurar servidor LAMP
  hosts: webservers
  become: yes
  
  vars:
    # Variables para Apache
    apache_listen_port: 80
    apache_create_vhosts: true
    apache_vhosts:
      - servername: "www.example.com"
        documentroot: "/var/www/html"
    
    # Variables para PHP
    php_packages:
      - php
      - php-mysql
      - php-curl
      - php-gd
    
    # Variables para MySQL
    mysql_root_password: "supersecret"
    mysql_databases:
      - name: myapp_db
    mysql_users:
      - name: myapp_user
        password: "secret"
        priv: "myapp_db.*:ALL"
  
  roles:
    - geerlingguy.apache
    - geerlingguy.php
    - geerlingguy.mysql
```

#### 3. Ejecutar

```bash
ansible-playbook -i hosts.cfg webserver.yml
```

### Organización con roles de Galaxy y propios

```
proyecto/
├── ansible.cfg
├── hosts.cfg
├── requirements.yml
├── playbooks/
│   ├── webservers.yml
│   ├── databases.yml
│   └── site.yml
└── roles/
    ├── geerlingguy.apache/      # Desde Galaxy
    ├── geerlingguy.php/          # Desde Galaxy
    ├── geerlingguy.mysql/        # Desde Galaxy
    ├── mi_rol_personalizado/     # Propio
    └── otro_rol/                 # Propio
```

## Ejemplo completo de Rol personalizado

### Crear rol NTP Server

```bash
ansible-galaxy init roles/ntp_server
```

**`roles/ntp_server/tasks/main.yml`:**
```yaml
---
# tasks file for ntp_server

- name: Instalar chrony
  apt:
    name: chrony
    state: latest
    update_cache: yes

- name: Configurar chrony en servidor
  template:
    src: chrony.conf.j2
    dest: /etc/chrony/chrony.conf
    owner: root
    group: root
    mode: '0644'
  notify: Reiniciar chrony

- name: Asegurar que chrony está en ejecución
  service:
    name: chrony
    state: started
    enabled: yes
```

**`roles/ntp_server/templates/chrony.conf.j2`:**
```jinja2
# Servidores NTP upstream
pool 2.debian.pool.ntp.org offline iburst

# Servidor NTP local
server {{ ntp_server }} iburst

# Permitir clientes de la red de gestión
allow {{ management_network }}/24

# Archivos de configuración
keyfile /etc/chrony/chrony.keys
commandkey 1
driftfile /var/lib/chrony/chrony.drift
logdir /var/log/chrony

# Configuraciones adicionales
maxupdateskew 100.0
dumponexit
dumpdir /var/lib/chrony
logchange 0.5
hwclockfile /etc/adjtime
rtcsync
```

**`roles/ntp_server/defaults/main.yml`:**
```yaml
---
# defaults file for ntp_server
ntp_server: 1.es.pool.ntp.org
management_network: 10.0.0.0
```

**`roles/ntp_server/handlers/main.yml`:**
```yaml
---
# handlers file for ntp_server

- name: Reiniciar chrony
  service:
    name: chrony
    state: restarted
```

**`roles/ntp_server/meta/main.yml`:**
```yaml
---
galaxy_info:
  author: Tu Nombre
  description: Configuración de servidor NTP con Chrony
  license: MIT
  min_ansible_version: 2.9
  
  platforms:
    - name: Ubuntu
      versions:
        - focal
        - jammy
  
  galaxy_tags:
    - ntp
    - chrony
    - time

dependencies: []
```

**Usar el rol:**
```yaml
---
- name: Configurar servidor NTP
  hosts: controller
  become: yes
  vars:
    ntp_server: pool.ntp.org
    management_network: 10.0.0.0
  roles:
    - ntp_server
```

## Ejemplo: Despliegue de aplicación PHP desde GitHub

### Crear rol

```bash
ansible-galaxy init roles/php_app_deploy
```

**`roles/php_app_deploy/tasks/main.yml`:**
```yaml
---
- name: Actualizar cache de paquetes
  apt:
    update_cache: yes

- name: Instalar Apache y PHP
  apt:
    name:
      - apache2
      - php
      - libapache2-mod-php
      - git
    state: present

- name: Clonar repositorio de la aplicación
  git:
    repo: '{{ app_repo_url }}'
    dest: '{{ app_install_path }}'
    version: '{{ app_version }}'
    force: yes

- name: Configurar permisos
  file:
    path: '{{ app_install_path }}'
    owner: www-data
    group: www-data
    recurse: yes

- name: Cambiar puerto de Apache
  lineinfile:
    path: /etc/apache2/ports.conf
    regexp: '^Listen 80'
    line: 'Listen {{ app_port }}'
  notify: Reiniciar Apache

- name: Cambiar puerto en VirtualHost
  lineinfile:
    path: /etc/apache2/sites-enabled/000-default.conf
    regexp: '^<VirtualHost \*:80>'
    line: '<VirtualHost *:{{ app_port }}>'
  notify: Reiniciar Apache

- name: Asegurar que Apache está en ejecución
  service:
    name: apache2
    state: started
    enabled: yes
```

**`roles/php_app_deploy/defaults/main.yml`:**
```yaml
---
app_repo_url: 'https://github.com/usuario/app.git'
app_version: 'main'
app_install_path: /var/www/html/app
app_port: 8080
```

**`roles/php_app_deploy/handlers/main.yml`:**
```yaml
---
- name: Reiniciar Apache
  service:
    name: apache2
    state: restarted
```

**Playbook:**
```yaml
---
- name: Desplegar aplicación PHP
  hosts: appservers
  become: yes
  vars:
    app_repo_url: 'https://github.com/ualmtorres/diariostic.git'
    app_port: 8080
  roles:
    - php_app_deploy
```

## Dependencias entre Roles

Los roles pueden depender de otros roles usando el archivo `meta/main.yml`:

**`roles/webserver/meta/main.yml`:**
```yaml
---
dependencies:
  - role: common
  - role: firewall
    firewall_allowed_ports:
      - 80
      - 443
```

Cuando ejecutas el rol `webserver`, primero se ejecutarán `common` y `firewall`.

## Buenas prácticas para Roles

1. **Un propósito por rol:** Cada rol debe tener una responsabilidad clara
2. **Usar defaults:** Proporcionar valores por defecto sensatos
3. **Documentar:** Mantener README.md actualizado con variables y uso
4. **Idempotencia:** Asegurar que el rol se puede ejecutar múltiples veces
5. **Testing:** Probar el rol en entornos de test
6. **Variables claras:** Usar nombres descriptivos con prefijos
7. **Metadatos completos:** Llenar correctamente meta/main.yml
8. **Control de versiones:** Usar Git para versionar roles
9. **Separar lógica:** No mezclar instalación con configuración si no es necesario
10. **Reutilización:** Diseñar para que sea fácilmente reutilizable

### Ejemplo de README.md para un rol

```markdown
# Rol: Apache

Instala y configura el servidor web Apache.

## Requisitos

- Ubuntu 20.04 o superior
- Ansible 2.9 o superior

## Variables

### Defaults (defaults/main.yml)

- `apache_port`: Puerto de escucha (default: 80)
- `max_clients`: Máximo número de clientes (default: 200)
- `server_name`: Nombre del servidor (default: localhost)

### Variables requeridas

Ninguna. Todas las variables tienen valores por defecto.

## Dependencias

Ninguna.

## Ejemplo de uso

```yaml
- hosts: webservers
  become: yes
  roles:
    - role: apache
      apache_port: 8080
      server_name: www.example.com
```

## Handlers

- `Reiniciar Apache`: Reinicia el servicio apache2
- `Recargar Apache`: Recarga la configuración sin reiniciar

## Autor

Tu Nombre (tu.email@example.com)

## Licencia

MIT
```

## Resumen

- Los **roles** organizan y modularizanla configuración de Ansible
- Estructura predefinida: tasks, handlers, templates, files, vars, defaults, meta
- **Ansible Galaxy** es el repositorio de roles de la comunidad
- Comandos clave: `ansible-galaxy init`, `install`, `search`, `list`
- Los roles se pueden **reutilizar** en múltiples proyectos
- Usar `requirements.yml` para gestionar dependencias
- Los roles pueden tener **dependencias** de otros roles
- **Buenas prácticas:** un propósito por rol, documentación, idempotencia
- La organización en roles facilita el **mantenimiento y escalabilidad**

---
- Usar variables y plantillas para personalizar comportamientos.
- Compartir roles a través de Ansible Galaxy o repositorios internos.
- Versionar los roles y probarlos de forma aislada.

El uso de roles es esencial para proyectos de automatización grandes y colaborativos.
