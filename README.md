# k8


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

        kubeadm join 192.168.20.11:6443 --token xgeh1h.3rd9utsmxtestmd6 \
        --discovery-token-ca-cert-hash sha256:514ac9cb307f943568b46ef2b3fb2e97a76b6907adc0f0451ec056f22e283e67

3. instalar calico 
       kubectl apply -f https://raw.githubusercontent.com/projectcalico/calico/v3.25.0/manifests/calico.yaml

4. Intalar metalb  para crear un load balancer  para que salga por ahi la conexion hacie l exterior de los pods   

      kubectl apply -f https://raw.githubusercontent.com/metallb/metallb/main/config/manifests/metallb-native.yaml

      #creamos el archivo para el loadbalancer 
        apiVersion: metallb.io/v1beta1
        kind: IPAddressPool
        metadata:
        name: salida-internet-pool
        namespace: metallb-system
        spec:
        addresses:
            - 192.168.20.18
        ---
        apiVersion: metallb.io/v1beta1
        kind: L2Advertisement
        metadata:
        name: salida-internet-advert
        namespace: metallb-system
