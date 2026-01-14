## Diagrama básico de funcionamiento (ASCII)

```
	   +-------------------+
	   |  Controlador      |
	   |  (Ansible)        |
	   +-------------------+
			   |
	   +---------+---------+
	   |         |         |
   SSH  |     SSH |     SSH |
	   v         v         v
 +-----------+ +-----------+ +-----------+
 | Nodo 1    | | Nodo 2    | | Nodo 3    |
 | (Servidor)| | (Servidor)| | (Servidor)|
 +-----------+ +-----------+ +-----------+
```

El controlador Ansible se conecta por SSH a los nodos gestionados para aplicar configuraciones y tareas.
# 1. Introducción a Ansible

Ansible es una herramienta de automatización de TI de código abierto que permite gestionar configuraciones, desplegar aplicaciones y orquestar tareas de manera sencilla y eficiente. Se basa en un enfoque sencillo, sin agentes y utiliza SSH para comunicarse con los nodos gestionados.

## Ventajas de Ansible
- Fácil de aprender y usar.
- No requiere instalar agentes en los nodos.
- Basado en YAML, lo que facilita la lectura y escritura de configuraciones.
- Escalable y flexible.

## Modelo declarativo vs. imperativo
- **Declarativo:** Se describe el estado deseado del sistema y Ansible se encarga de alcanzarlo.
- **Imperativo:** Se indican los pasos exactos a seguir para llegar al estado deseado.

Ansible utiliza principalmente el modelo declarativo, facilitando la gestión de infraestructuras complejas.

## ¿Cómo funciona Ansible?
Ansible se basa en una arquitectura simple:

- **Controlador:** Es el equipo desde el que se ejecutan los comandos de Ansible. Aquí se instalan los playbooks y la configuración.
- **Nodos gestionados:** Son los servidores o dispositivos sobre los que Ansible actúa. No requieren que se instale ningún software adicional, solo acceso SSH y Python.

El controlador se conecta a los nodos gestionados mediante SSH y ejecuta las tareas definidas en los playbooks.

## Ejemplo de flujo de trabajo
1. El usuario escribe un playbook en YAML.
2. El controlador ejecuta el playbook.
3. Ansible se conecta por SSH a los nodos gestionados.
4. Se aplican los cambios necesarios para alcanzar el estado deseado.

## Diagrama básico de funcionamiento

![Diagrama básico de Ansible](https://raw.githubusercontent.com/ansible/ansible-assets/main/ansible-diagram.png)

> **Figura:** El controlador Ansible se conecta por SSH a los nodos gestionados para aplicar configuraciones.

---