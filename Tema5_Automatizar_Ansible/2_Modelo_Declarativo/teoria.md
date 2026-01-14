# 2. Modelo Declarativo de Ansible


# Modelo Declarativo en Ansible

El modelo declarativo es un enfoque en el que se describe el estado final deseado de los sistemas gestionados, en lugar de detallar los pasos exactos para llegar a ese estado. Ansible, como herramienta de automatización, utiliza este modelo para simplificar la gestión de infraestructuras.

## ¿Qué significa "declarativo"?
En el modelo declarativo, el usuario especifica **qué** quiere conseguir, no **cómo** conseguirlo. Por ejemplo, se indica que un paquete debe estar instalado, pero no se detallan los comandos para instalarlo, comprobar si ya está, etc. Ansible se encarga de analizar el estado actual y aplicar solo los cambios necesarios para alcanzar el estado deseado.

## Idempotencia
Una de las características clave del modelo declarativo es la **idempotencia**: ejecutar varias veces el mismo playbook no produce efectos secundarios si el sistema ya está en el estado deseado. Esto permite aplicar configuraciones de forma segura y repetida.

## Ventajas del modelo declarativo
- **Simplicidad:** El usuario se centra en el resultado, no en los pasos.
- **Mantenimiento sencillo:** Los playbooks son más fáciles de leer, entender y modificar.
- **Menos errores:** Al no tener que gestionar los estados intermedios, se reducen los fallos por condiciones no previstas.
- **Reproducibilidad:** Se puede aplicar la misma configuración en diferentes entornos con resultados consistentes.

## Diferencias con el modelo imperativo
- **Imperativo:** El usuario indica paso a paso cómo alcanzar el objetivo (por ejemplo, scripts bash).
- **Declarativo:** El usuario describe el estado final y la herramienta decide los pasos necesarios.

## Ejemplo simple de playbook declarativo
```yaml
- hosts: servidores
  tasks:
    - name: Asegurar que Apache esté instalado
      apt:
        name: apache2
        state: present
```
En este ejemplo, Ansible se asegura de que Apache esté instalado. Si ya lo está, no hace nada; si no, lo instala. No importa el estado previo, el resultado será siempre el mismo.

## Otros ejemplos de tareas declarativas
- Asegurar que un archivo existe o no existe.
- Garantizar que un servicio está iniciado o detenido.
- Definir permisos y propietarios de archivos.

El modelo declarativo es fundamental para la automatización moderna y la gestión de infraestructuras como código (IaC).