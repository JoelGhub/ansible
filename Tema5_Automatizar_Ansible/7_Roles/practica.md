
# Práctica: Organización y uso de roles en Ansible

## Objetivo
Aprender a crear, descargar y utilizar roles locales y de Ansible Galaxy, así como pasar variables a los roles.

## Pasos

### 1. Crear un rol local con Ansible Galaxy
```bash
ansible-galaxy init rol_apache
```
Esto generará la estructura básica del rol en la carpeta `rol_apache/`.

### 2. Añadir tareas al rol local
Edita el archivo `rol_apache/tasks/main.yml` y añade:
```yaml
- name: Instalar Apache
  apt:
    name: apache2
    state: present
- name: Asegurar que Apache está iniciado
  service:
    name: apache2
    state: started
```

### 3. Crear un playbook que use el rol local
Crea un archivo llamado `usar_rol_apache.yml`:
```yaml
- hosts: web
  become: yes
  roles:
    - rol_apache
```

### 4. Descargar e instalar un rol de Ansible Galaxy
Por ejemplo, el rol oficial de nginx:
```bash
ansible-galaxy install geerlingguy.nginx
```

### 5. Crear un playbook que use un rol de Galaxy
Crea un archivo llamado `usar_rol_nginx.yml`:
```yaml
- hosts: web
  become: yes
  roles:
    - geerlingguy.nginx
```

### 6. Llamar varios roles y pasar variables
Crea un archivo llamado `multi_roles.yml`:
```yaml
- hosts: web
  become: yes
  roles:
    - { role: geerlingguy.nginx, nginx_vhosts: [{server_name: "miweb.local", root: "/var/www/html"}] }
    - rol_apache
```

### 7. Ejecutar los playbooks
```bash
ansible-playbook -i inventario.ini usar_rol_apache.yml
ansible-playbook -i inventario.ini usar_rol_nginx.yml
ansible-playbook -i inventario.ini multi_roles.yml
```

---

Esta práctica te permitirá dominar la organización, reutilización y personalización de configuraciones usando roles locales y de Ansible Galaxy.