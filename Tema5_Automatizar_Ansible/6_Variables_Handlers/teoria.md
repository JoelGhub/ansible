# 6. Variables y Handlers en Ansible

## Introducción

Las **variables** y **handlers** son herramientas fundamentales para crear playbooks flexibles, reutilizables y eficientes en Ansible.

- **Variables:** Permiten parametrizar y personalizar la configuración
- **Handlers:** Permiten ejecutar tareas solo cuando son necesarias (cuando algo cambia)

## Variables en Ansible

### ¿Qué son las Variables?

Las variables en Ansible permiten almacenar valores que se pueden reutilizar en playbooks, plantillas y tareas. Hacen que los playbooks sean más flexibles y mantenibles.

Las variables se gestionan mediante el motor de plantillas **Jinja2**, que proporciona sustitución de variables usando la sintaxis de doble llave: `{{ variable }}`

### Sintaxis básica

```yaml
# Definir variable
vars:
  app_name: mi_aplicacion
  app_port: 8080

# Usar variable
tasks:
  - name: Mostrar información
    debug:
      msg: "La aplicación {{ app_name }} usa el puerto {{ app_port }}"
```

### Tipos de variables

#### 1. Variables de playbook
Definidas directamente en la sección `vars` del playbook.

```yaml
---
- name: Ejemplo con variables locales
  hosts: webservers
  vars:
    http_port: 80
    max_clients: 200
    server_name: www.example.com
  tasks:
    - name: Mostrar configuración
      debug:
        msg: "Servidor {{ server_name }} en puerto {{ http_port }}"
```

#### 2. Variables de archivo (`vars_files`)
Cargadas desde archivos externos.

**Archivo `variables.yml`:**
```yaml
---
username: johndoe
fullname: John Doe
app_port: 8080
database_name: myapp_db
```

**Playbook:**
```yaml
---
- name: Usar variables de archivo
  hosts: all
  vars_files:
    - variables.yml
  tasks:
    - name: Mostrar usuario
      debug:
        msg: "Usuario: {{ username }}, Nombre: {{ fullname }}"
```

#### 3. Variables en `group_vars/` y `host_vars/`

Las variables definidas en `group_vars/all.yml` son visibles para todos los playbooks del directorio sin necesidad de incluirlas explícitamente.

**Estructura de directorios:**
```
proyecto/
├── ansible.cfg
├── hosts.cfg
├── playbook.yml
├── group_vars/
│   ├── all.yml          # Variables para todos los grupos
│   ├── webservers.yml   # Variables para grupo webservers
│   └── databases.yml    # Variables para grupo databases
└── host_vars/
    ├── web1.yml         # Variables para host específico
    └── db1.yml
```

**Ejemplo `group_vars/all.yml`:**
```yaml
---
manager: { name: manager, ip: 20.0.1.7 }
managed_1: { name: managed-1, ip: 20.0.1.11 }
managed_2: { name: managed-2, ip: 20.0.1.4 }

ntp_server: pool.ntp.org
dns_servers:
  - 8.8.8.8
  - 8.8.4.4
```

**Playbook que usa estas variables:**
```yaml
---
- name: Configurar hosts
  hosts: all
  tasks:
    - name: Añadir entrada a /etc/hosts
      lineinfile:
        path: /etc/hosts
        line: "{{ manager.ip }} {{ manager.name }}"
```

#### 4. Variables de inventario

Se definen en el archivo de inventario a nivel de host o grupo.

```ini
[webservers]
web1 ansible_host=192.168.1.10 http_port=80
web2 ansible_host=192.168.1.11 http_port=8080

[webservers:vars]
ansible_user=ubuntu
domain=example.com
```

#### 5. Variables desde línea de comandos (`--extra-vars` o `-e`)

Tienen la **máxima prioridad** y sobrescriben cualquier otra variable.

```bash
# Pasar variables como pares clave=valor
ansible-playbook playbook.yml -e "app_version=2.0 environment=production"

# Pasar variables como JSON
ansible-playbook playbook.yml -e '{"app_version":"2.0", "environment":"production"}'

# Pasar variables desde archivo
ansible-playbook playbook.yml -e @variables.json
```

#### 6. Variables solicitadas durante la ejecución (`vars_prompt`)

```yaml
---
- name: Solicitar información al usuario
  hosts: localhost
  vars_prompt:
    - name: username
      prompt: "¿Cuál es tu nombre de usuario?"
      private: no
    
    - name: password
      prompt: "¿Cuál es tu contraseña?"
      private: yes    # No mostrar al escribir
  
  tasks:
    - name: Mostrar nombre de usuario
      debug:
        msg: "Hola {{ username }}"
```

#### 7. Variables Facts (hechos del sistema)

Ansible recopila automáticamente información de los hosts (gather_facts: yes por defecto).

```yaml
---
- name: Usar facts del sistema
  hosts: all
  tasks:
    - name: Mostrar información del sistema
      debug:
        msg: |
          Hostname: {{ ansible_hostname }}
          OS: {{ ansible_distribution }} {{ ansible_distribution_version }}
          IP: {{ ansible_default_ipv4.address }}
          CPU: {{ ansible_processor_cores }}
          RAM: {{ ansible_memtotal_mb }} MB
```

**Acceso a facts:**
```yaml
# Ambas formas son válidas
{{ ansible_hostname }}
{{ ansible_facts['hostname'] }}

# Usar punto o corchetes para navegar
{{ ansible_facts.hostname }}
{{ ansible_facts['default_ipv4']['address'] }}
```

**Ver todos los facts de un host:**
```bash
ansible hostname -m setup
```

#### 8. Variables registradas (`register`)

Captura la salida de un módulo para usar en tareas posteriores.

```yaml
---
- name: Usar register
  hosts: localhost
  tasks:
    - name: Obtener fecha actual
      command: date
      register: fecha_actual
      changed_when: false
    
    - name: Mostrar resultado
      debug:
        var: fecha_actual
    
    - name: Usar stdout
      debug:
        msg: "La fecha es: {{ fecha_actual.stdout }}"
    
    - name: Verificar si comando fue exitoso
      debug:
        msg: "Comando ejecutado correctamente"
      when: fecha_actual.rc == 0
```

### Precedencia de variables (de menor a mayor)

Ansible tiene un orden de precedencia para las variables. Las variables con mayor precedencia sobrescriben a las de menor precedencia.

**Orden de precedencia (de menor a mayor):**

1. Valores por defecto de roles (`role defaults`)
2. Variables de inventario o script de inventario (`inventory file` o `script group vars`)
3. Archivo `inventory group_vars/all`
4. Archivo `playbook group_vars/all`
5. Archivo `inventory group_vars/*`
6. Archivo `playbook group_vars/*`
7. Variables de inventario o script de host (`inventory file` o `script host vars`)
8. Archivo `inventory host_vars/*`
9. Archivo `playbook host_vars/*`
10. Facts del host / set_facts cacheados
11. Variables del play (`vars`)
12. Variables solicitadas (`vars_prompt`)
13. Archivos de variables (`vars_files`)
14. Variables de rol (`role vars`)
15. Variables de bloque (`block vars`)
16. Variables de tarea (`task vars`)
17. `include_vars`
18. `set_facts` / variables registradas
19. Parámetros de rol (`role params`) e `include params`
20. **Variables extra (`extra vars`)** - **MÁXIMA PRIORIDAD**

### Operaciones con variables

#### Concatenación de strings

```yaml
vars:
  base_path: /opt
  app_name: myapp
  
tasks:
  - name: Crear ruta completa
    debug:
      msg: "Ruta: {{ base_path }}/{{ app_name }}"
```

#### Valores por defecto

```yaml
tasks:
  - name: Usar valor por defecto
    debug:
      msg: "Puerto: {{ http_port | default(80) }}"
```

#### Filtros Jinja2

```yaml
tasks:
  - name: Convertir a mayúsculas
    debug:
      msg: "{{ app_name | upper }}"
  
  - name: Generar hash de contraseña
    user:
      name: deploy
      password: "{{ 'mypassword' | password_hash('sha512') }}"
  
  - name: Obtener primeros 8 caracteres
    debug:
      msg: "{{ long_string | truncate(8) }}"
```

### Bucles con variables

#### Loop simple

```yaml
---
- name: Crear múltiples usuarios
  hosts: all
  become: yes
  vars:
    usuarios:
      - alice
      - bob
      - charlie
  tasks:
    - name: Crear usuarios
      user:
        name: "{{ item }}"
        state: present
      loop: "{{ usuarios }}"
```

#### Loop con diccionarios

```yaml
---
- name: Crear usuarios con información completa
  hosts: all
  become: yes
  vars:
    usuarios:
      - { name: 'alice', group: 'developers', shell: '/bin/bash' }
      - { name: 'bob', group: 'admins', shell: '/bin/zsh' }
      - { name: 'charlie', group: 'developers', shell: '/bin/bash' }
  tasks:
    - name: Crear usuarios
      user:
        name: "{{ item.name }}"
        group: "{{ item.group }}"
        shell: "{{ item.shell }}"
        state: present
      loop: "{{ usuarios }}"
```

#### Iteración con `with_items` (sintaxis antigua)

```yaml
# Sintaxis antigua (aún funciona pero se prefiere loop)
tasks:
  - name: Instalar paquetes
    apt:
      name: "{{ item }}"
      state: present
    with_items:
      - vim
      - git
      - curl
```

### Uso de variables en templates

**Archivo de template `app.conf.j2`:**
```jinja2
[application]
app_name = {{ app_name }}
app_port = {{ app_port }}
debug = {{ debug_mode }}

[database]
host = {{ db_host }}
port = {{ db_port }}
name = {{ db_name }}
user = {{ db_user }}

[servers]
{% for server in web_servers %}
server{{ loop.index }} = {{ server }}
{% endfor %}
```

**Playbook:**
```yaml
---
- name: Desplegar configuración
  hosts: appservers
  vars:
    app_name: MiAplicacion
    app_port: 8080
    debug_mode: false
    db_host: db.example.com
    db_port: 3306
    db_name: myapp_db
    db_user: app_user
    web_servers:
      - web1.example.com
      - web2.example.com
      - web3.example.com
  tasks:
    - name: Desplegar configuración
      template:
        src: app.conf.j2
        dest: /etc/app/app.conf
```

## Handlers en Ansible

### ¿Qué son los Handlers?

Los **handlers** son tareas especiales que se ejecutan **solo cuando son notificadas** por otras tareas. Son ideales para acciones que solo deben ocurrir si hay cambios reales, como:

- Reiniciar servicios
- Recargar configuraciones
- Reconstruir índices
- Limpiar cachés

### Características de los Handlers

1. Se ejecutan **solo si una tarea los notifica** con `notify`
2. Se ejecutan **al final del play**, no inmediatamente
3. Se ejecutan **solo una vez** aunque sean notificados múltiples veces
4. Se ejecutan **en el orden en que se definen**, no en el orden de notificación

### Ejemplo básico de Handler

```yaml
---
- name: Configurar Apache
  hosts: webservers
  become: yes
  tasks:
    - name: Instalar Apache
      apt:
        name: apache2
        state: present
    
    - name: Copiar configuración de Apache
      copy:
        src: apache2.conf
        dest: /etc/apache2/apache2.conf
      notify: Reiniciar Apache    # Notifica al handler
    
    - name: Copiar sitio web
      copy:
        src: index.html
        dest: /var/www/html/index.html
      notify: Reiniciar Apache
  
  handlers:
    - name: Reiniciar Apache    # Nombre debe coincidir con notify
      service:
        name: apache2
        state: restarted
```

**Comportamiento:**
- Si los archivos no cambian → Handler **NO** se ejecuta
- Si uno o ambos archivos cambian → Handler se ejecuta **una sola vez** al final

### Múltiples Handlers

```yaml
---
- name: Configurar servicios
  hosts: webservers
  become: yes
  tasks:
    - name: Modificar configuración de Apache
      template:
        src: apache.conf.j2
        dest: /etc/apache2/apache2.conf
      notify:
        - Validar configuración Apache
        - Reiniciar Apache
    
    - name: Modificar configuración de PHP
      template:
        src: php.ini.j2
        dest: /etc/php/7.4/apache2/php.ini
      notify: Reiniciar Apache
  
  handlers:
    - name: Validar configuración Apache
      command: apache2ctl configtest
    
    - name: Reiniciar Apache
      service:
        name: apache2
        state: restarted
```

### Forzar ejecución de Handlers con `meta: flush_handlers`

Por defecto, los handlers se ejecutan al final del play. Puedes forzar su ejecución inmediata:

```yaml
---
- name: Configuración con flush
  hosts: webservers
  become: yes
  tasks:
    - name: Actualizar configuración crítica
      copy:
        src: critical.conf
        dest: /etc/app/critical.conf
      notify: Reiniciar servicio
    
    - name: Forzar ejecución de handlers ahora
      meta: flush_handlers
    
    - name: Tarea que requiere que el servicio esté reiniciado
      command: /usr/local/bin/check_service.sh
  
  handlers:
    - name: Reiniciar servicio
      service:
        name: mi_servicio
        state: restarted
```

### Handlers condicionales

```yaml
handlers:
  - name: Reiniciar Apache
    service:
      name: apache2
      state: restarted
    when: ansible_distribution == "Ubuntu"
```

### Ejemplo completo: Configuración de servidor web

```yaml
---
- name: Configurar servidor web completo
  hosts: webservers
  become: yes
  vars:
    http_port: 80
    max_clients: 200
    admin_email: admin@example.com
  
  tasks:
    - name: Instalar Apache y módulos
      apt:
        name:
          - apache2
          - libapache2-mod-php
        state: present
        update_cache: yes
    
    - name: Configurar Apache
      template:
        src: templates/apache2.conf.j2
        dest: /etc/apache2/apache2.conf
        validate: 'apache2ctl -t -f %s'
      notify:
        - Validar Apache
        - Recargar Apache
    
    - name: Configurar sitio virtual
      template:
        src: templates/vhost.conf.j2
        dest: /etc/apache2/sites-available/000-default.conf
      notify: Recargar Apache
    
    - name: Habilitar módulos de Apache
      apache2_module:
        name: "{{ item }}"
        state: present
      loop:
        - rewrite
        - ssl
        - headers
      notify: Reiniciar Apache
    
    - name: Copiar archivos del sitio web
      copy:
        src: website/
        dest: /var/www/html/
        owner: www-data
        group: www-data
        mode: '0644'
    
    - name: Asegurar que Apache está en ejecución
      service:
        name: apache2
        state: started
        enabled: yes
  
  handlers:
    - name: Validar Apache
      command: apache2ctl configtest
      changed_when: false
    
    - name: Recargar Apache
      service:
        name: apache2
        state: reloaded
    
    - name: Reiniciar Apache
      service:
        name: apache2
        state: restarted
```

## Buenas prácticas

### Variables

1. **Nombres descriptivos:** Usa nombres claros y consistentes
   ```yaml
   # ❌ Evitar
   p: 8080
   
   # ✅ Preferir
   application_port: 8080
   ```

2. **Usar archivos separados:** Para proyectos grandes, separar variables en archivos
3. **Proteger datos sensibles:** Usar Ansible Vault para contraseñas y secretos
4. **Documentar variables:** Añadir comentarios explicando el propósito
5. **Valores por defecto:** Proporcionar valores por defecto sensatos
6. **Evitar sobre-uso:** No todo necesita ser variable

### Handlers

1. **Nombres descriptivos:** El nombre del handler debe describir la acción
2. **Usar handlers para operaciones costosas:** Reiniciar servicios, recargar configuraciones
3. **Un handler por acción:** No agrupar múltiples acciones en un handler
4. **Validar antes de reiniciar:** Verificar configuración antes de aplicar
5. **Handlers idempotentes:** Asegurar que se pueden ejecutar múltiples veces

## Resumen

- Las **variables** parametrizan y hacen flexibles los playbooks
- Se pueden definir en múltiples lugares con diferentes precedencias
- **Extra vars** tiene la máxima prioridad
- **Facts** proporcionan información automática del sistema
- **Register** captura salidas para usar posteriormente
- Los **handlers** se ejecutan solo cuando son notificados
- Los handlers se ejecutan **al final** y **solo una vez**
- Usar `notify` para activar handlers
- Usar `meta: flush_handlers` para ejecución inmediata

---
