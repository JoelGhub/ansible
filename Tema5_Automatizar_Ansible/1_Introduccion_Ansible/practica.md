# Práctica: Instalación de VirtualBox, creación de VM y primeros pasos con Ansible

## 1. Instalación de VirtualBox

### En macOS
1. Descarga el instalador desde: https://www.virtualbox.org/wiki/Downloads
2. Ejecuta el archivo .dmg descargado
3. Sigue el asistente de instalación
4. **Importante:** Puede que necesites autorizar la extensión del kernel en Preferencias del Sistema > Seguridad y Privacidad
5. Verifica la instalación abriendo VirtualBox desde Aplicaciones

### En Linux (Ubuntu/Debian)
```bash
sudo apt update
sudo apt install virtualbox
# Opcional: instalar Extension Pack para funcionalidades adicionales
wget https://download.virtualbox.org/virtualbox/7.0.6/Oracle_VM_VirtualBox_Extension_Pack-7.0.6.vbox-extpack
sudo vboxmanage extpack install Oracle_VM_VirtualBox_Extension_Pack-7.0.6.vbox-extpack
```

### En Windows
1. Descarga el instalador desde la web oficial
2. Ejecuta como administrador
3. Sigue el asistente (acepta instalar drivers cuando se solicite)
4. Reinicia si es necesario

### Verificación de la instalación
```bash
vboxmanage --version
```

---

## 2. Creación de máquinas virtuales para el laboratorio

### Descarga de la imagen ISO
1. Descarga Ubuntu Server 22.04 LTS desde: https://ubuntu.com/download/server
2. Guarda el archivo .iso en una carpeta fácil de localizar

### Creación de la VM principal (Controlador Ansible)
1. Abre VirtualBox y haz clic en "Nueva"
2. **Nombre:** ansible-controller
3. **Tipo:** Linux
4. **Versión:** Ubuntu (64-bit)
5. **Memoria:** 2048 MB (2 GB)
6. **Disco duro:**
   - Crear un disco duro virtual ahora
   - Tipo: VDI (VirtualBox Disk Image)
   - Asignación: Dinámicamente asignado
   - Tamaño: 20 GB
7. **Configurar red:**
   - Ir a Configuración > Red
   - Adaptador 1: NAT (para acceso a internet)
   - Adaptador 2: Red interna (nombre: "ansible-lab")

### Instalación del sistema operativo
1. Selecciona la VM y haz clic en "Iniciar"
2. Selecciona el archivo .iso de Ubuntu cuando se solicite
3. **Durante la instalación:**
   - Idioma: Español (o el que prefieras)
   - Diseño de teclado: Spanish
   - Tipo de instalación: Ubuntu Server (minimal)
   - Usuario: ansible
   - Contraseña: ansible123 (o la que prefieras)
   - **¡Importante!** Marca "Install OpenSSH server" cuando aparezca la opción
4. Completa la instalación y reinicia

### Creación de VM adicionales (Nodos gestionados)
Repite el proceso anterior para crear 2 VMs adicionales:
- **Nombres:** ansible-node1, ansible-node2
- **Configuración:** Igual que la anterior pero con 1 GB RAM
- **Usuarios:** node1/node123 y node2/node123 respectivamente

---

## 3. Instalación y configuración de Ansible

### Instalación en el controlador (ansible-controller)

#### Método 1: Mediante PPA (recomendado)
```bash
sudo apt update
sudo apt install software-properties-common
sudo add-apt-repository --yes --update ppa:ansible/ansible
sudo apt install ansible
```

#### Método 2: Mediante pip
```bash
sudo apt update
sudo apt install python3 python3-pip
pip3 install ansible
echo 'export PATH="$HOME/.local/bin:$PATH"' >> ~/.bashrc
source ~/.bashrc
```

### Verificación de la instalación
```bash
ansible --version
ansible-config --version
```

### Configuración inicial
```bash
# Crear directorio para configuraciones de Ansible
mkdir -p ~/ansible-lab
cd ~/ansible-lab

# Crear archivo de configuración ansible.cfg
cat > ansible.cfg << EOF
[defaults]
host_key_checking = False
inventory = inventory.ini
remote_user = ansible
[ssh_connection]
ssh_args = -o ControlMaster=auto -o ControlPersist=60s
EOF
```

## 4. Configuración del entorno de trabajo

### Obtener las IPs de las VMs
```bash
# En cada VM, ejecutar para obtener la IP:
ip addr show
# Anota las IPs de la red interna (normalmente 192.168.x.x)
```

### Crear el inventario
```bash
# Crear archivo de inventario
cat > inventory.ini << EOF
[webservers]
ansible-node1 ansible_host=192.168.56.101
ansible-node2 ansible_host=192.168.56.102

[all:vars]
ansible_user=node1
ansible_ssh_pass=node123
EOF
```

### Configurar SSH sin contraseña (recomendado)
```bash
# Generar clave SSH
ssh-keygen -t rsa -b 4096 -f ~/.ssh/id_rsa -N ""

# Copiar la clave a los nodos (ejecutar para cada nodo)
ssh-copy-id node1@192.168.56.101
ssh-copy-id node2@192.168.56.102

# Actualizar inventario para usar claves SSH
cat > inventory.ini << EOF
[webservers]
ansible-node1 ansible_host=192.168.56.101 ansible_user=node1
ansible-node2 ansible_host=192.168.56.102 ansible_user=node2
EOF
```

## 5. Primeros comandos y ejemplos básicos

### Verificar conectividad
```bash
# Ping a todos los nodos
ansible all -m ping

# Debería mostrar algo como:
# ansible-node1 | SUCCESS => {
#     "changed": false,
#     "ping": "pong"
# }
```

### Comandos ad-hoc básicos
```bash
# Obtener información del sistema
ansible all -m setup --limit ansible-node1

# Ejecutar comandos simples
ansible all -a "uptime"
ansible all -a "df -h"
ansible webservers -a "free -m"

# Instalar un paquete
ansible all -b -m apt -a "name=htop state=present"
# (-b significa "become", es decir, usar sudo)
```

### Primer playbook básico
```bash
# Crear primer playbook
cat > primer-playbook.yml << EOF
---
- name: Mi primer playbook
  hosts: all
  become: true
  tasks:
    - name: Actualizar cache de paquetes
      apt:
        update_cache: yes
    
    - name: Instalar paquetes básicos
      apt:
        name:
          - htop
          - curl
          - wget
        state: present
    
    - name: Crear un archivo de prueba
      file:
        path: /tmp/ansible-test.txt
        state: touch
        mode: '0644'
EOF

# Ejecutar el playbook
ansible-playbook primer-playbook.yml
```

### Verificar el resultado
```bash
# Comprobar que los paquetes se instalaron
ansible all -a "which htop"

# Comprobar que el archivo se creó
ansible all -a "ls -la /tmp/ansible-test.txt"
```

## 6. Ejercicios adicionales

1. **Exploración:** Ejecuta `ansible all -m setup` y explora la información que proporciona sobre cada nodo.

2. **Gestión de archivos:** Crea un playbook que copie un archivo de configuración desde el controlador a todos los nodos.

3. **Servicios:** Instala y configura nginx en todos los nodos usando Ansible.

4. **Variables:** Modifica el playbook para usar variables que definan qué paquetes instalar.

## 7. Reflexión
- ¿Qué ventajas observas al usar Ansible frente a conectarse por SSH manualmente a cada servidor?
- ¿Cómo crees que escalaría este proceso con 100 servidores en lugar de 2?
- ¿Qué problemas podrían surgir en un entorno real y cómo los solucionarías?

---

¡Ya tienes tu entorno completo y has ejecutado tus primeros comandos con Ansible!

---

<br><br><br><br><br><br><br><br><br><br>

# SOLUCIONES A LOS EJERCICIOS ADICIONALES

## Solución Ejercicio 1: Exploración con setup

```bash
# Ejecutar setup en un nodo específico
ansible ansible-node1 -m setup

# Filtrar información específica
ansible all -m setup -a "filter=ansible_distribution*"
ansible all -m setup -a "filter=ansible_memtotal_mb"
ansible all -m setup -a "filter=ansible_processor*"

# Guardar la información en un archivo para análisis
ansible all -m setup > facts.json
```

**Información útil que puedes encontrar:**
- Sistema operativo y versión
- Memoria total y disponible
- Información del procesador
- Direcciones IP e interfaces de red
- Espacio en disco
- Variables de entorno

## Solución Ejercicio 2: Gestión de archivos

```bash
# Crear un archivo de configuración local
cat > mi-config.conf << EOF
# Mi archivo de configuración personalizada
server_name=mi-servidor
debug_mode=true
max_connections=100
timeout=30
EOF

# Crear playbook para copiar archivos
cat > gestion-archivos.yml << EOF
---
- name: Gestión de archivos de configuración
  hosts: all
  become: true
  tasks:
    - name: Crear directorio de configuración
      file:
        path: /etc/mi-app
        state: directory
        mode: '0755'
    
    - name: Copiar archivo de configuración
      copy:
        src: mi-config.conf
        dest: /etc/mi-app/mi-config.conf
        mode: '0644'
        backup: yes
    
    - name: Crear archivo desde template
      copy:
        content: |
          # Archivo generado por Ansible
          hostname={{ ansible_hostname }}
          ip={{ ansible_default_ipv4.address }}
          fecha={{ ansible_date_time.date }}
        dest: /tmp/info-sistema.txt
        mode: '0644'
EOF

# Ejecutar el playbook
ansible-playbook gestion-archivos.yml
```

## Solución Ejercicio 3: Instalación y configuración de nginx

```bash
# Crear playbook para nginx
cat > nginx-setup.yml << EOF
---
- name: Instalación y configuración de nginx
  hosts: all
  become: true
  tasks:
    - name: Instalar nginx
      apt:
        name: nginx
        state: present
        update_cache: yes
    
    - name: Crear página web personalizada
      copy:
        content: |
          <!DOCTYPE html>
          <html>
          <head>
              <title>Servidor {{ ansible_hostname }}</title>
          </head>
          <body>
              <h1>¡Hola desde {{ ansible_hostname }}!</h1>
              <p>IP: {{ ansible_default_ipv4.address }}</p>
              <p>Configurado con Ansible</p>
          </body>
          </html>
        dest: /var/www/html/index.html
        mode: '0644'
    
    - name: Iniciar y habilitar nginx
      systemd:
        name: nginx
        state: started
        enabled: true
    
    - name: Verificar que nginx está corriendo
      service_facts:
    
    - name: Mostrar estado del servicio
      debug:
        msg: "Nginx está {{ ansible_facts.services['nginx.service'].state }}"
EOF

# Ejecutar el playbook
ansible-playbook nginx-setup.yml

# Verificar que funciona
ansible all -a "curl -s http://localhost"
```

## Solución Ejercicio 4: Uso de variables

```bash
# Crear archivo de variables
cat > variables.yml << EOF
---
paquetes_basicos:
  - htop
  - curl
  - wget
  - vim
  - git

paquetes_desarrollo:
  - build-essential
  - python3-pip
  - nodejs
  - npm

usuario_app: "appuser"
directorio_app: "/opt/mi-aplicacion"
EOF

# Crear playbook con variables
cat > playbook-con-variables.yml << EOF
---
- name: Playbook usando variables
  hosts: all
  become: true
  vars_files:
    - variables.yml
  vars:
    mensaje_bienvenida: "¡Servidor configurado con Ansible!"
  
  tasks:
    - name: Instalar paquetes básicos
      apt:
        name: "{{ paquetes_basicos }}"
        state: present
        update_cache: yes
    
    - name: Instalar paquetes de desarrollo
      apt:
        name: "{{ paquetes_desarrollo }}"
        state: present
      when: inventory_hostname == "ansible-node1"
    
    - name: Crear usuario de aplicación
      user:
        name: "{{ usuario_app }}"
        shell: /bin/bash
        create_home: yes
    
    - name: Crear directorio de aplicación
      file:
        path: "{{ directorio_app }}"
        state: directory
        owner: "{{ usuario_app }}"
        mode: '0755'
    
    - name: Crear archivo con mensaje personalizado
      copy:
        content: |
          {{ mensaje_bienvenida }}
          Configurado el: {{ ansible_date_time.date }}
          En el servidor: {{ ansible_hostname }}
        dest: "{{ directorio_app }}/bienvenida.txt"
        owner: "{{ usuario_app }}"
        mode: '0644'
    
    - name: Mostrar información de variables
      debug:
        msg: 
          - "Paquetes básicos: {{ paquetes_basicos | join(', ') }}"
          - "Usuario creado: {{ usuario_app }}"
          - "Directorio: {{ directorio_app }}"
EOF

# Ejecutar el playbook
ansible-playbook playbook-con-variables.yml

# Verificar resultados
ansible all -a "ls -la /opt/mi-aplicacion/"
ansible all -a "cat /opt/mi-aplicacion/bienvenida.txt"
```

### Ejemplo avanzado con variables por grupo

```bash
# Crear inventario con variables por grupo
cat > inventory-avanzado.ini << EOF
[webservers]
ansible-node1 ansible_host=192.168.56.101 ansible_user=node1

[databases]  
ansible-node2 ansible_host=192.168.56.102 ansible_user=node2

[webservers:vars]
tipo_servidor=web
puerto_app=80
paquetes_especiales=["apache2", "php"]

[databases:vars]
tipo_servidor=database
puerto_app=3306
paquetes_especiales=["mysql-server", "mysql-client"]

[all:vars]
entorno=desarrollo
EOF

# Playbook que usa variables por grupo
cat > playbook-grupos.yml << EOF
---
- name: Configuración específica por tipo de servidor
  hosts: all
  become: true
  tasks:
    - name: Mostrar tipo de servidor
      debug:
        msg: "Este es un servidor de tipo: {{ tipo_servidor }}"
    
    - name: Instalar paquetes específicos del grupo
      apt:
        name: "{{ paquetes_especiales }}"
        state: present
        update_cache: yes
      when: paquetes_especiales is defined
    
    - name: Configurar firewall para el puerto específico
      debug:
        msg: "Configurando firewall para puerto {{ puerto_app }}"
EOF

# Usar el inventario avanzado
ansible-playbook -i inventory-avanzado.ini playbook-grupos.yml
```