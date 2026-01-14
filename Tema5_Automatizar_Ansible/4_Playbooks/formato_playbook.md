# Formato básico de un Playbook en Ansible

Un playbook es un archivo YAML que describe una o varias tareas a ejecutar sobre uno o varios hosts. Es la unidad principal de automatización en Ansible.

## Estructura básica
```yaml
- name: Nombre descriptivo del play
  hosts: grupo_o_host
  become: yes  # (opcional) para ejecutar como root
  vars:        # (opcional) variables para el play
    variable1: valor1
  tasks:
    - name: Descripción de la tarea
      módulo:
        argumento1: valor
        argumento2: valor
  handlers:    # (opcional) tareas que se ejecutan solo si son notificadas
    - name: Nombre del handler
      módulo:
        argumento: valor
  roles:       # (opcional) roles a incluir
    - nombre_rol
```

## Instrucciones principales y para qué sirven
- `name`: Descripción legible del play o tarea.
- `hosts`: Define los hosts o grupos objetivo.
- `become`: Escala privilegios (por ejemplo, a root).
- `vars`: Define variables locales al play.
- `tasks`: Lista de tareas a ejecutar.
- `handlers`: Tareas especiales que se ejecutan solo cuando son notificadas.
- `roles`: Incluye roles para modularizar la configuración.
- `gather_facts`: Recoge información del sistema (por defecto en yes).
- `tags`: Permite etiquetar tareas para ejecución selectiva.

## Ejemplo 1: Playbook mínimo
```yaml
- hosts: web
  tasks:
    - name: Comprobar conectividad
      ping:
```

## Ejemplo 2: Playbook con variables y handlers
```yaml
- name: Desplegar página web
  hosts: web
  become: yes
  vars:
    pagina: "¡Hola desde Ansible!"
  tasks:
    - name: Instalar Apache
      apt:
        name: apache2
        state: present
    - name: Crear página de inicio
      copy:
        content: "{{ pagina }}"
        dest: /var/www/html/index.html
      notify: Reiniciar Apache
  handlers:
    - name: Reiniciar Apache
      service:
        name: apache2
        state: restarted
```

## Ejemplo 3: Playbook con roles
```yaml
- hosts: web
  become: yes
  roles:
    - rol_apache
```

## Más información
- Documentación oficial: https://docs.ansible.com/ansible/latest/user_guide/playbooks.html
