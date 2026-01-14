# Práctica: Comprendiendo el modelo declarativo en Ansible

## Objetivo
Aplicar el modelo declarativo de Ansible asegurando el estado de un servicio y un archivo en una máquina virtual.

## Pasos

### 1. Crear el archivo de inventario
Crea un archivo llamado `inventario.ini` con el siguiente contenido (ajusta la IP según tu entorno):

```
[web]
192.168.56.101
```

### 2. Crear un playbook para instalar y arrancar Apache
Crea un archivo llamado `apache.yml` con el siguiente contenido:

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


### 3. Ejecutar el playbook
Lanza el playbook con:
```bash
ansible-playbook -i inventario.ini apache.yml
```


### 4. Comprobar idempotencia
Ejecuta el playbook varias veces y observa que, si el estado ya es el deseado, Ansible no realiza cambios adicionales.


### 5. Modificar el playbook para gestionar un archivo
A continuación, añade una tarea para asegurar que existe un archivo `/tmp/prueba.txt`:

```yaml
    - name: Crear archivo de prueba
      file:
        path: /tmp/prueba.txt
        state: touch
```


### 6. Vuelve a ejecutar el playbook
Comprueba que el archivo se crea si no existe y que no se modifica si ya está presente.

---

Esta práctica te ayudará a entender cómo Ansible aplica el modelo declarativo y la importancia de la idempotencia en la automatización.