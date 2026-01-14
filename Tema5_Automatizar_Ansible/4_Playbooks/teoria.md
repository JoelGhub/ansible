# 4. Playbooks en Ansible


Un playbook es un archivo en formato YAML que describe un conjunto de tareas a ejecutar sobre uno o varios hosts. Es la unidad principal de automatización en Ansible y permite definir configuraciones, despliegues y orquestación de servicios de forma estructurada y repetible.

## Sintaxis y estructura de un playbook
Un playbook está compuesto por una lista de "plays". Cada play define:
- Los hosts objetivo.
- Las variables a utilizar.
- Las tareas a ejecutar.
- Los handlers y roles asociados.

### Ejemplo básico de playbook
```yaml
- name: Instalar y arrancar Apache
  hosts: web
  become: yes
  tasks:
    - name: Instalar Apache
      apt:
        name: apache2
        state: present
    - name: Asegurar que Apache esté iniciado
      service:
        name: apache2
        state: started
```

## Componentes principales
- **hosts:** Define los hosts o grupos sobre los que se ejecutarán las tareas.
- **tasks:** Lista de acciones a realizar (instalar paquetes, copiar archivos, etc).
- **vars:** Variables específicas para el play.
- **handlers:** Tareas que se ejecutan solo cuando son notificadas.
- **roles:** Permite incluir roles para organizar la configuración.

## Buenas prácticas
- Usar nombres descriptivos para las tareas.
- Comentar el código para facilitar el mantenimiento.
- Reutilizar código mediante includes y roles.
- Separar variables y handlers en archivos independientes si el playbook crece.

## Ejemplo avanzado con variables y handlers
```yaml
- name: Desplegar aplicación web
  hosts: web
  become: yes
  vars:
    paquete: apache2
    archivo_index: /var/www/html/index.html
  tasks:
    - name: Instalar Apache
      apt:
        name: "{{ paquete }}"
        state: present
      notify: Reiniciar Apache
    - name: Copiar página de inicio
      copy:
        content: "¡Bienvenido a mi web!"
        dest: "{{ archivo_index }}"
  handlers:
    - name: Reiniciar Apache
      service:
        name: apache2
        state: restarted
```

## Includes y reutilización
Se pueden incluir tareas, handlers o variables desde otros archivos usando `include_tasks`, `import_tasks` o `vars_files`.

Los playbooks son la base para la automatización avanzada y la gestión de infraestructuras como código.