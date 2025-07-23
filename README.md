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
            - 192.168.20.18/32
        ---
        apiVersion: metallb.io/v1beta1
        kind: L2Advertisement
        metadata:
        name: l2
        namespace: metallb-system

5. Instalar el ingress
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

     

   
   
