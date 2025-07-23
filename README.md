# k8

0 #0preparacion de isntalacion para el nodo master como para el Worker

     sudo hostnamectl set-hostname worker-node-1
 Paso 2: Desactivar Swap y habilitar módulos de Kubernetes
    sudo swapoff -a
    sudo sed -i '/ swap / s/^/#/' /etc/fstab

    sudo modprobe overlay
    sudo modprobe br_netfilter

      sudo tee /etc/sysctl.d/k8s.conf <<EOF
      net.bridge.bridge-nf-call-iptables = 1
      net.ipv4.ip_forward = 1
      net.bridge.bridge-nf-call-arptables = 1
      EOF

      sudo sysctl --system

      #anular de froma permante en el arranque
      nano /etc/fstab


      # cambiar repositorio apra isntalar 
      deb http://archive.ubuntu.com/ubuntu/ jammy main universe restricted multiverse
      deb http://archive.ubuntu.com/ubuntu/ jammy-updates main universe restricted multiverse
      deb http://archive.ubuntu.com/ubuntu/ jammy-backports main universe restricted multiverse
      deb http://archive.ubuntu.com/ubuntu/ jammy-security main universe restricted multiverse
      
      sudo apt-get update
      
      sudo apt update && sudo apt install -y curl apt-transport-https ca-certificates software-properties-common

      sudo apt install -y containerd

      sudo mkdir -p /etc/containerd
      containerd config default | sudo tee /etc/containerd/config.toml 

      Edita el archivo y cambia SystemdCgroup = false a true en containerd/config.toml:

      sudo nano /etc/containerd/config.toml

      sudo systemctl restart containerd
      sudo systemctl enable containerd

    #instalamos kuberntees
    curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.29/deb/Release.key | sudo tee /etc/apt/keyrings/kubernetes-apt-keyring.asc
      echo "deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.asc] https://pkgs.k8s.io/core:/stable:/v1.29/deb/ /" | sudo tee /etc/apt/sources.list.d/kubernetes.list
      sudo apt update


sudo apt install -y kubelet kubeadm kubectl
sudo apt-mark hold kubelet kubeadm kubectl


sudo systemctl enable --now kubelet

 



#instalacion de Nodo master de Kubernetes
1.  una vez de  haber instalado  el basico para los nodos
   sudo hostnamectl set-hostname master-node

    sudo modprobe br_netfilter
sudo tee /etc/sysctl.d/k8s.conf <<EOF
net.bridge.bridge-nf-call-iptables = 1
net.ipv4.ip_forward = 1
net.bridge.bridge-nf-call-arptables = 1
EOF


    sudo sysctl --system
 2. 
       sudo kubeadm init --pod-network-cidr=192.168.0.0/16

        mkdir -p $HOME/.kube
        sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
        sudo chown $(id -u):$(id -g) $HOME/.kube/config

        
    
    kubeadm join 192.168.20.15:6443 --token rt95m4.c23b2rhokoqev1x2 \
        --discovery-token-ca-cert-hash sha256:4ea2533cbb84a91165d40ce40ec1f1f0ab5546f826f7c47602eadd3b243a7c3a

4. instalar calico
     Antes de isntalar calico debes de tener como minimo un worker par aue calico pueda isntalarse
       kubectl apply -f https://raw.githubusercontent.com/projectcalico/calico/v3.25.0/manifests/calico.yaml



                  Y obtuviste:
               
               perl
               Copiar
               Editar
               WARN/ERROR: connect: no such file or directory "/var/run/dockershim.sock"
               ...
               kube-apiserver ... Running
               🔍 ¿Qué significa esto?
               El kube-apiserver está corriendo correctamente, lo cual es buena señal. ✅
               
               Pero crictl está intentando conectarse a un socket de Docker (dockershim.sock) que ya no existe, porque:
               
               dockershim fue removido desde Kubernetes 1.24.
               
               Tu sistema probablemente usa containerd, no Docker.
               
               ✅ ¿Cómo solucionarlo?
               1. 🛠 Establecer el socket correcto para crictl
               Edita (o crea) el archivo de configuración para crictl:
               
               bash
               Copiar
               Editar
               sudo nano /etc/crictl.yaml
               Agrega el siguiente contenido (asumiendo que usas containerd):
               
               yaml
               Copiar
               Editar
               runtime-endpoint: unix:///run/containerd/containerd.sock
               Guarda y cierra con CTRL+O, ENTER, CTRL+X.
               
               Luego prueba:
               
               bash
               Copiar
               Editar
               sudo crictl ps
               Ya no deberías ver errores.
               
               

6. Intalar metalb  para crear un load balancer  para que salga por ahi la conexion hacie l exterior de los pods   

      kubectl apply -f https://raw.githubusercontent.com/metallb/metallb/main/config/manifests/metallb-native.yaml

      #creamos el archivo para el loadbalancer 
        apiVersion: metallb.io/v1beta1
        kind: IPAddressPool
        metadata:
        name: salida-internet-pool
        namespace: metallb-system
        spec:
        addresses:
            - 192.168.20.18/32
        ---
        apiVersion: metallb.io/v1beta1
        kind: L2Advertisement
        metadata:
        name: l2
        namespace: metallb-system

   aplicar
  ------------------------------------------------------

   PASOS PARA INSTALAR HELM (Versión estable)
🔹 1. Agrega el repositorio oficial de Helm
bash
Copiar
Editar
curl https://baltocdn.com/helm/signing.asc | sudo apt-key add -
sudo apt-get install apt-transport-https --yes
echo "deb https://baltocdn.com/helm/stable/debian/ all main" | sudo tee /etc/apt/sources.list.d/helm-stable-debian.list
🔹 2. Actualiza e instala Helm
bash
Copiar
Editar
sudo apt update
sudo apt install helm -y
🔹 3. Verifica la instalación
bash
Copiar
Editar
helm version
*****************************************************************************************

8. Instalar el ingress
     kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.9.5/deploy/static/provider/cloud/deploy.yaml
   
      comandos para revisar
         kubectl get svc -n ingress-nginx
      Si no lo hace automáticamente como LoadBalancer, modifica el servicio manualmente
        # patch-ingress.yaml
          spec:
         loadBalancerIP: 192.168.10.81


                1. Instalar ingress-nginx con Helm
            Si aún no lo tienes instalado con Helm, este es el comando recomendado para un clúster en Proxmox con LoadBalancer (que hace salida a Internet):
            
            bash
            Copiar
            Editar
            helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
            helm repo update
            
            helm install ingress-nginx ingress-nginx/ingress-nginx \
              --namespace ingress-nginx \
              --create-namespace \
              --set controller.service.type=LoadBalancer \
              --set controller.ingressClassResource.name=nginx \
              --set controller.ingressClassResource.controllerValue="k8s.io/ingress-nginx"
            ✅ Asegúrate de que el LoadBalancer (MetalLB, HAProxy, etc.) esté funcionando correctamente.

   
 6.Configurar HTTPS con Let's Encrypt (cert-manager)
   1.	Instala cert-manager:
       kubectl apply -f https://github.com/cert-manager/cert-manager/releases/latest/download/cert-manager.yaml
     	
   3.	Crea un ClusterIssuer para Let's Encrypt:****
      
      # cluster-issuer.yaml
       apiVersion: cert-manager.io/v1
       kind: ClusterIssuer
       metadata:
         name: letsencrypt-prod
       spec:
         acme:
           email: tu-email@dominio.com
           server: https://acme-v02.api.letsencrypt.org/directory
           privateKeySecretRef:
             name: letsencrypt-prod
           solvers:
           - http01:
               ingress:
                 class: nginx

      #aplicar kubectl apply -f cluster-issuer.yaml

7.para proxmox debemos d einstalr helm para los certificados

  helm install cert-manager jetstack/cert-manager \
  --namespace cert-manager \
  --create-namespace \
  --set installCRDs=true \
  --set "extraArgs={--dns01-recursive-nameservers-only,--dns01-recursive-nameservers=8.8.8.8:53}"

     

   
   
