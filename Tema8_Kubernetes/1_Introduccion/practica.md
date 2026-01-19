# Práctica: Instalación de Kubernetes en VirtualBox con Ubuntu Server

En esta práctica crearemos un cluster de Kubernetes **real** usando VirtualBox con 2 máquinas Ubuntu Server. Esto te dará una experiencia mucho más cercana a un entorno de producción.

## 1. Requisitos previos

- **VirtualBox** instalado
- **CPU:** Al menos 4 cores  
- **RAM:** Mínimo 8GB
- **Espacio en disco:** 40GB disponibles
- **Conexión a Internet**

## 2. Instalación de VirtualBox

### En macOS
```bash
# Descargar desde https://www.virtualbox.org/wiki/Downloads
# O con Homebrew
brew install --cask virtualbox
```

### En Linux (Ubuntu/Debian)
```bash
sudo apt update
sudo apt install virtualbox virtualbox-ext-pack
```

### Verificar
```bash
vboxmanage --version
```

## 3. Descargar Ubuntu Server

Descarga Ubuntu Server 22.04 LTS: https://ubuntu.com/download/server

## 4. Crear las máquinas virtuales

### VM 1 - k8s-master (Control Plane)
1. VirtualBox > Nueva
2. **Nombre:** k8s-master  
3. **Tipo:** Linux / Ubuntu (64-bit)
4. **Memoria:** 2048 MB
5. **Disco:** 20 GB (VDI, dinámico)
6. **Red:**
   - Adaptador 1: NAT (internet)
   - Adaptador 2: Red interna (nombre: "k8s-network")

### VM 2 - k8s-worker1
- Igual configuración que el master
- **Nombre:** k8s-worker1

## 5. Instalar Ubuntu Server

En cada VM:
1. Iniciar > Seleccionar ISO de Ubuntu
2. **Usuario master:** `master` / Pass: `master123`
3. **Usuario worker:** `worker` / Pass: `worker123`  
4. **Importante:** Marcar "Install OpenSSH server"
5. Completar instalación y reiniciar

## 6. Configuración inicial

### En CADA VM:

```bash
# Actualizar sistema
sudo apt update && sudo apt upgrade -y

# Instalar herramientas
sudo apt install -y curl wget vim net-tools

# Deshabilitar swap (requerido por K8s)
sudo swapoff -a
sudo sed -i '/ swap / s/^/#/' /etc/fstab
```

### Configurar IPs estáticas

**En k8s-master:**
```bash
sudo vim /etc/netplan/00-installer-config.yaml
```

```yaml
network:
  version: 2
  ethernets:
    enp0s3:
      dhcp4: true
    enp0s8:
      addresses:
        - 192.168.56.10/24
      dhcp4: no
```

**En k8s-worker1:**
```yaml
network:
  version: 2
  ethernets:
    enp0s3:
      dhcp4: true
    enp0s8:
      addresses:
        - 192.168.56.11/24
      dhcp4: no
```

```bash
# Aplicar
sudo netplan apply
ip addr show
```

### Configurar /etc/hosts en AMBAS VMs:

```bash
sudo bash -c 'cat >> /etc/hosts << EOF
192.168.56.10 k8s-master
192.168.56.11 k8s-worker1
EOF'

# Probar conectividad
ping -c 3 k8s-master
ping -c 3 k8s-worker1
```

## 7. Opción A: k3s (RECOMENDADO - Más fácil)

k3s es Kubernetes ligero, perfecto para aprender.

### En k8s-master:

```bash
# Instalar k3s
curl -sfL https://get.k3s.io | sh -

# Verificar
sudo systemctl status k3s

# Obtener token para workers
sudo cat /var/lib/rancher/k3s/server/node-token

# Configurar kubectl
mkdir -p ~/.kube
sudo cp /etc/rancher/k3s/k3s.yaml ~/.kube/config
sudo chown $USER:$USER ~/.kube/config

# Instalar kubectl
sudo snap install kubectl --classic

# Verificar
kubectl get nodes
```

### En k8s-worker1:

```bash
# Unir al cluster (reemplaza TOKEN con el del master)
curl -sfL https://get.k3s.io | K3S_URL=https://192.168.56.10:6443 \
  K3S_TOKEN=<TOKEN_DEL_MASTER> sh -

# Verificar
sudo systemctl status k3s-agent
```

### Verificar cluster:

```bash
# Desde el master
kubectl get nodes
# Deberías ver ambos nodos en Ready
```

## 8. Opción B: kubeadm (Más educativo)

kubeadm es la instalación estándar de Kubernetes.

### En AMBAS VMs:

```bash
# Módulos del kernel
cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
overlay
br_netfilter
EOF

sudo modprobe overlay
sudo modprobe br_netfilter

# Parámetros de red
cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-iptables  = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.ipv4.ip_forward                 = 1
EOF

sudo sysctl --system

# Instalar containerd
sudo apt-get update
sudo apt-get install -y containerd
sudo mkdir -p /etc/containerd
containerd config default | sudo tee /etc/containerd/config.toml
sudo sed -i 's/SystemdCgroup = false/SystemdCgroup = true/' /etc/containerd/config.toml
sudo systemctl restart containerd
sudo systemctl enable containerd

# Instalar kubeadm, kubelet, kubectl
sudo apt-get install -y apt-transport-https ca-certificates curl
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.28/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.28/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list
sudo apt-get update
sudo apt-get install -y kubelet kubeadm kubectl
sudo apt-mark hold kubelet kubeadm kubectl
```

### Solo en k8s-master:

```bash
# Inicializar cluster
sudo kubeadm init --pod-network-cidr=10.244.0.0/16 --apiserver-advertise-address=192.168.56.10

# GUARDA el comando kubeadm join que aparece

# Configurar kubectl
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config

# Instalar red (Flannel)
kubectl apply -f https://github.com/flannel-io/flannel/releases/latest/download/kube-flannel.yml

# Verificar
kubectl get nodes
```

### Solo en k8s-worker1:

```bash
# Ejecutar el comando kubeadm join del master
sudo kubeadm join 192.168.56.10:6443 --token <token> \
    --discovery-token-ca-cert-hash sha256:<hash>
```

## 9. Primeros pasos

### Crear un Pod:

```bash
kubectl run nginx --image=nginx:latest
kubectl get pods
kubectl get pods -o wide
kubectl describe pod nginx
kubectl delete pod nginx
```

### Crear un Deployment:

```bash
kubectl create deployment web --image=nginx:latest --replicas=3
kubectl get pods -o wide
kubectl scale deployment web --replicas=5
kubectl get pods
```

### Exponer como servicio:

```bash
kubectl expose deployment web --port=80 --type=NodePort
kubectl get svc

# Probar (usa el NODEPORT que te muestre)
curl 192.168.56.10:<NODEPORT>
curl 192.168.56.11:<NODEPORT>
```

## 10. Trabajar con YAML

```bash
mkdir ~/k8s-lab
cd ~/k8s-lab

cat > app.yaml << 'EOF'
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mi-app
spec:
  replicas: 4
  selector:
    matchLabels:
      app: mi-app
  template:
    metadata:
      labels:
        app: mi-app
    spec:
      containers:
      - name: nginx
        image: nginx:1.21
        ports:
        - containerPort: 80
---
apiVersion: v1
kind: Service
metadata:
  name: mi-app-svc
spec:
  selector:
    app: mi-app
  type: NodePort
  ports:
  - port: 80
    targetPort: 80
EOF

kubectl apply -f app.yaml
kubectl get all
```

## 11. Comandos esenciales

```bash
# Ver recursos
kubectl get nodes
kubectl get pods
kubectl get deployments
kubectl get services
kubectl get all

# Detalles
kubectl describe node k8s-master
kubectl describe pod <pod-name>

# Logs
kubectl logs <pod-name>
kubectl logs <pod-name> -f

# Ejecutar
kubectl exec -it <pod-name> -- bash

# Eliminar
kubectl delete pod <pod-name>
kubectl delete deployment <name>
kubectl delete -f app.yaml
```

## 12. Ejercicios

1. **Desplegar app multi-réplica:** Crea un deployment con 6 réplicas y observa la distribución
2. **Escalar:** Escala de 2 a 10 réplicas y observa el proceso
3. **Rolling update:** Actualiza la imagen y observa la actualización gradual
4. **Debug:** Crea un pod que falle y practiza troubleshooting

## 13. k3s vs kubeadm

| Característica | k3s | kubeadm |
|---------------|-----|---------|
| Instalación | 1 comando | Varios pasos |
| Tamaño | ~50MB | ~2GB |
| RAM | ~512MB | ~1.5GB |
| Ideal para | Aprendizaje, edge | Producción, certs |

**Recomendación:** Empieza con k3s, luego prueba kubeadm para certificaciones.

## 14. Solución de problemas

### Nodos no Ready:
```bash
kubectl get pods -n kube-system
# Para k3s
sudo systemctl status k3s
# Para kubeadm
kubectl get componentstatuses
```

### Sin conectividad entre pods:
```bash
kubectl get pods -n kube-system | grep flannel
kubectl rollout restart daemonset kube-flannel-ds -n kube-system
```

## 15. Limpieza

```bash
# Eliminar recursos
kubectl delete deployments --all
kubectl delete services --all

# Desinstalar k3s
sudo /usr/local/bin/k3s-uninstall.sh  # master
sudo /usr/local/bin/k3s-agent-uninstall.sh  # worker

# Resetear kubeadm
sudo kubeadm reset
```

---

