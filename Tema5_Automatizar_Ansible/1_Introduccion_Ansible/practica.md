# Práctica: Instalación de VirtualBox, creación de una VM y Ansible

## 1. Instalación de VirtualBox

### En macOS
Descarga el instalador desde la web oficial:
https://www.virtualbox.org/wiki/Downloads

Ejecuta el instalador y sigue los pasos en pantalla.

### En Linux (Ubuntu/Debian)
```bash
sudo apt update
sudo apt install virtualbox
```

### En Windows
Descarga el instalador desde la web oficial y sigue el asistente.

---

## 2. Creación de una máquina virtual en VirtualBox

1. Abre VirtualBox y haz clic en "Nueva".
2. Asigna un nombre a la VM (por ejemplo, "ansible-vm").
3. Selecciona el tipo de sistema operativo (por ejemplo, Linux, Ubuntu 64-bit).
4. Asigna memoria RAM (mínimo 1024 MB recomendado).
5. Crea un disco duro virtual (VDI, tamaño dinámico, al menos 10 GB).
6. Inicia la VM y monta una ISO de Ubuntu/Debian para instalar el sistema operativo.
7. Sigue el asistente de instalación del sistema operativo dentro de la VM.

---

## 3. Instalación de Ansible en la VM

### En Ubuntu/Debian
```bash
sudo apt update
sudo apt install software-properties-common
sudo add-apt-repository --yes --update ppa:ansible/ansible
sudo apt install ansible
```

### Verificar instalación
```bash
ansible --version
```

---

¡Ya tienes tu entorno listo para comenzar a practicar con Ansible!