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
    - 192.168.10.81/32
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



curl https://baltocdn.com/helm/signing.asc | gpg --dearmor | sudo tee /usr/share/keyrings/helm.gpg > /dev/null
sudo apt-get install apt-transport-https --yes
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/helm.gpg] https://baltocdn.com/helm/stable/debian/ all main" | sudo tee /etc/apt/sources.list.d/helm-stable-debian.list
sudo apt-get update
sudo apt-get install helm

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
    email: grupo90pr@gmail.com
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


importante :
kubectl -n kube-system edit configmap coredns
    hosts {
        192.168.20.18 balanceo.facturameya.online
        fallthrough
    }




example:
Crea un archivo coredns-wildcard.yaml y aplícalo con kubectl apply -f … para evitar el texto con \n.

yaml
Copiar
Editar
apiVersion: v1
kind: ConfigMap
metadata:
  name: coredns
  namespace: kube-system
data:
  Corefile: |
    .:53 {
        errors
        health
        ready

        # Split-DNS: TODO cambia 192.168.20.18 por tu IP interna del Ingress
        # El hosts de CoreDNS acepta wildcard en la 1ª etiqueta (*.dominio)
        hosts {
            192.168.20.18 *.facturameya.online
            fallthrough
        }

        kubernetes cluster.local in-addr.arpa ip6.arpa {
            pods insecure
            fallthrough
        }
        prometheus :9153
        forward . /etc/resolv.conf
        cache 30
        loop
        reload
        loadbalance
    }
Aplica y reinicia CoreDNS:

bash
Copiar
Editar
kubectl apply -f coredns-wildcard.yaml

kubectl -n kube-system rollout restart deploy coredns

2️⃣ Comprobar resolución de DNS desde dentro del clúster
Abre un pod temporal para hacer pruebas:

bash
Copiar
Editar
kubectl run -it --rm --image=busybox dns-test --restart=Never -- sh
Dentro del pod:

sh
Copiar
Editar
nslookup balanceo.facturameya.online
nslookup otro.facturameya.online
Ambos deberían devolver 192.168.20.18 (o la IP interna que configuraste).

********************************************************************************************************************************************





8 instalar metrics server 
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml

kubectl get pods -n kube-system

: Habilitar opciones en metrics-server
Algunas veces metrics-server no funciona por problemas de certificado SSL. Para solucionarlo, edita la configuración:
1️⃣ Abre el archivo de despliegue de metrics-server:


kubectl edit deployment metrics-server -n kube-system
2️⃣ Busca la sección command: dentro del contenedor metrics-server y agrégale estos flags:


spec:
  containers:
  - name: metrics-server
    args:
    - --kubelet-insecure-tls
    - --kubelet-preferred-address-types=InternalIP
3️⃣ Guarda los cambios y reinicia los pods:


            kubectl delete pod -n kube-system -l k8s-app=metrics-server
Esto hará que los pods se reinicien con la nueva configuración.
________________________________________
🛠️ Paso 3: Probar metrics-server
Después de la instalación, espera unos segundos y ejecuta:

      kubectl top nodes
Si funciona correctamente, verás algo como:
scss

NAME           CPU(cores)   MEMORY(bytes)
master-node    250m        500Mi


8.1 instalar el dashboard


kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.7.0/aio/deploy/recommended.yaml

2. Crear un usuario admin para acceder
Crea un archivo llamado admin-user.yaml con este contenido:

apiVersion: v1
kind: ServiceAccount
metadata:
  name: admin-user
  namespace: kubernetes-dashboard
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: admin-user
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
- kind: ServiceAccount
  name: admin-user
  namespace: kubernetes-dashboard

  
Y aplícalo:


kubectl apply -f admin-user.yaml


3. Obtener el token de acceso
Obtén el token para acceder al Dashboard:


kubectl -n kubernetes-dashboard create token admin-user


Guarda este token, es el que usarás para iniciar sesión.




*************************************************************************************************************************************************************


9 Instalanmdo Longhorn 


helm repo add longhorn https://charts.longhorn.io
helm repo update
kubectl create namespace longhorn-system

helm install longhorn longhorn/longhorn \
  --namespace longhorn-system

kubectl -n longhorn-system get pods

✅ 4. Crear un PVC con acceso ReadWriteMany (RWX)
Aquí tienes un ejemplo YAML para crear un PVC RWX:

yaml
Copiar
Editar
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: shared-data
spec:
  accessModes:
    - ReadWriteMany
  storageClassName: longhorn
  resources:
    requests:
      storage: 5Gi
🧪 5. Probarlo con múltiples pods
Puedes montar ese PVC en múltiples pods como:

yaml
Copiar
Editar
apiVersion: v1
kind: Pod
metadata:
  name: writer-pod
spec:
  containers:
  - name: writer
    image: busybox
    command: ["sh", "-c", "echo hello > /mnt/data/hello.txt; sleep 3600"]
    volumeMounts:
    - mountPath: /mnt/data
      name: shared-vol
  volumes:
  - name: shared-vol
    persistentVolumeClaim:
      claimName: shared-data
Luego otro pod puede leer el mismo archivo usando el mismo PVC.



10. isntaldo  maridb galera alta disponibilidad
11. Modificación final del values.yaml con NodePort
yaml
Copiar
Editar
architecture: replication

auth:
  rootPassword: "sopa*806"
  replicationUser: repl
  replicationPassword: "sopa*807"
  database: ""
  username: ""
  password: ""

galera:
  enabled: true
  clusterBootstrap: true
  galeraClusterName: "mariadb-galera"

primary:
  persistence:
    enabled: true
    storageClass: longhorn
    size: 8Gi
  podAffinityPreset: soft
  nodeAffinityPreset:
    type: ""
  tolerations: []
  updateStrategy:
    type: RollingUpdate
  resources:
    requests:
      cpu: 250m
      memory: 512Mi
    limits:
      cpu: 500m
      memory: 1Gi

secondary:
  replicaCount: 2
  persistence:
    enabled: true
    storageClass: longhorn
    size: 8Gi
  resources:
    requests:
      cpu: 250m
      memory: 512Mi
    limits:
      cpu: 500m
      memory: 1Gi

service:
  type: NodePort
  nodePorts:
    mysql: 32006     # <-- Puedes cambiar este puerto si ya está ocupado
  port: 3306         # Puerto interno del contenedor
✅ Comando para instalar con este values.yaml
bash
Copiar
Editar
helm install mariadb-galera bitnami/mariadb-galera \
  --namespace mariadb --create-namespace \
  -f mariadb-galera-values.yaml

  # values-galera.yaml  (para bitnami/mariadb-galera)
replicaCount: 3

auth:
  rootPassword: "sopa*806"        # cambia en producción
  username: "appuser"             # opcional
  password: "app_pass"            # opcional
  database: "appdb"               # opcional

galera:
  name: "mariadb-galera"
  # el chart se auto-bootstrapea; no necesitas primary/secondary
  mariabackup:
    user: "backup"
    password: "backup_pass"

persistence:
  enabled: true
  storageClass: "longhorn"        # PVCs RWO (uno por pod)
  accessModes: ["ReadWriteOnce"]
  size: 8Gi

podAntiAffinityPreset: hard       # distribuye réplicas en nodos distintos
resources:
  requests: { cpu: "250m", memory: "512Mi" }
  limits:   { cpu: "500m", memory: "1Gi" }

service:
  type: ClusterIP
  ports:
    mysql: 3306


  ******************copair tus datos alos pods de galera
  1) Variables útiles
bash
Copiar
Editar
# Pod de Galera
POD=$(kubectl -n mariadb get pods -l app.kubernetes.io/name=mariadb-galera -o jsonpath='{.items[0].metadata.name}')

# Password root desde el Secret del chart
ROOTPW=$(kubectl -n mariadb get secret mariadb-galera -o jsonpath='{.data.mariadb-root-password}' | base64 -d)
2) Copia tus .sql al pod
bash
Copiar
Editar
kubectl -n mariadb exec -it "$POD" -- mkdir -p /tmp/sql
kubectl -n mariadb cp /home/lala/diag/sql/. "$POD":/tmp/sql
kubectl -n mariadb exec -it "$POD" -- ls -lh /tmp/sql
3) Crear las BDs e importar usando el binario correcto
bash
Copiar
Editar
kubectl -n mariadb exec -it "$POD" -- bash -lc '
set -e
MYSQL="/opt/bitnami/mariadb/bin/mariadb"     # usa este binario
PW="${MARIADB_ROOT_PASSWORD:-'"$ROOTPW"'}"

$MYSQL -uroot -p"$PW" -e "CREATE DATABASE IF NOT EXISTS almacen;"
$MYSQL -uroot -p"$PW" -e "CREATE DATABASE IF NOT EXISTS concurso_danza;"
$MYSQL -uroot -p"$PW" -e "CREATE DATABASE IF NOT EXISTS ecomerse;"
$MYSQL -uroot -p"$PW" -e "CREATE DATABASE IF NOT EXISTS planillas;"

$MYSQL -uroot -p"$PW" almacen        < /tmp/sql/almacen.sql
$MYSQL -uroot -p"$PW" concurso_danza < /tmp/sql/concurso_danza.sql
$MYSQL -uroot -p"$PW" ecomerse       < /tmp/sql/ecomerse.sql
$MYSQL -uroot -p"$PW" planillas      < /tmp/sql/planillas.sql
'
4) Verificar
bash
Copiar
Editar
kubectl -n mariadb exec -it "$POD" -- /opt/bitnami/mariadb/bin/mariadb -uroot -p"$ROOTPW" -e "S




   
   
