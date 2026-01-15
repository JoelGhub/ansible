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