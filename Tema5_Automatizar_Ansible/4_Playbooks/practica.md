# Práctica: Creación y ejecución de playbooks en Ansible

## Objetivo
Aprender a crear, estructurar y ejecutar playbooks con tareas, variables y handlers.

## Pasos

### 1. Crear un playbook básico
Crea un archivo llamado `instalar_apache.yml` con el siguiente contenido:

```yaml
- name: Instalar Apache y crear página de inicio
  hosts: web
  become: yes
  tasks:
    - name: Instalar Apache
      apt:
        name: apache2
        state: present
    - name: Crear página de inicio
      copy:
        content: "¡Hola desde Ansible!"
        dest: /var/www/html/index.html
```

### 2. Ejecutar el playbook
```bash
ansible-playbook -i inventario.ini instalar_apache.yml
```

### 3. Añadir un handler
Modifica el playbook para que reinicie Apache solo si la página de inicio cambia:

```yaml
    - name: Crear página de inicio
      copy:
        content: "¡Hola desde Ansible!"
        dest: /var/www/html/index.html
      notify: Reiniciar Apache
  handlers:
    - name: Reiniciar Apache
      service:
        name: apache2
        state: restarted
```

### 4. Probar la idempotencia
Ejecuta el playbook varias veces y observa que solo se reinicia Apache si hay cambios en la página.

---

Esta práctica te permitirá dominar la creación y ejecución de playbooks en Ansible.