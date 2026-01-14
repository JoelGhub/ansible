# Práctica: Uso de variables y handlers en Ansible

## Objetivo
Aprender a definir y utilizar variables y handlers en playbooks para automatizar tareas de forma eficiente.

## Pasos

### 1. Crear un playbook con variables
Crea un archivo llamado `variables_handlers.yml` con el siguiente contenido:

```yaml
- name: Práctica de variables y handlers
  hosts: web
  become: yes
  vars:
    paquete: nginx
    archivo_index: /var/www/html/index.html
  tasks:
    - name: Instalar Nginx
      apt:
        name: "{{ paquete }}"
        state: present
      notify: Reiniciar Nginx
    - name: Crear página de inicio
      copy:
        content: "¡Bienvenido a Nginx gestionado por Ansible!"
        dest: "{{ archivo_index }}"
      notify: Reiniciar Nginx
  handlers:
    - name: Reiniciar Nginx
      service:
        name: nginx
        state: restarted
```

### 2. Ejecutar el playbook
```bash
ansible-playbook -i inventario.ini variables_handlers.yml
```

### 3. Probar cambios en variables
Modifica el contenido de la página de inicio y vuelve a ejecutar el playbook para ver cómo se activa el handler solo si hay cambios.

---

Esta práctica te permitirá dominar el uso de variables y handlers en Ansible.