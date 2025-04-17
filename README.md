Instructions to deploy **YoPass** on Azure Kubernetes Service using the default **App Routing** add on
  1. Deploy AKS cluster through Azure portal.
  2. Create a namespace. ` kubectl create ns yopass `
  3. Create a tls secret named ` cert-tls ` which has the domain's certificate & private key by running below command. The domain's .crt & .key file should already be present.

     ```
     kubectl -n yopass create secret tls cert-tls --cert=domain_name.crt --key=domain_name.key
     ```
  4. Run the command ` kubectl -n app-routing-system get svc nginx ` to confirm if a **LoadBalancer** IP has been provisioned.

     ```
     pushkar [ ~ ]$ kubectl -n app-routing-system get svc nginx
     NAME    TYPE           CLUSTER-IP    EXTERNAL-IP    PORT(S)                      AGE
     nginx   LoadBalancer   10.0.58.180   20.174.45.64   80:32767/TCP,443:30598/TCP   110s
     ```
  5. Deploy the `memcached` & `yopass` deployment & service using the `kubectl` command

     ```
     kubectl -n yopass apply -f yopass-dep.yml -f yopass-svc.yml -f memcached-dep.yml -f memcached-svc.yml
     ```
  6. Put the FQDN for which the secret has been created in ` app-routing-ingress.yml ` file and then run the command ` kubectl -n yopass apply -f app-routing-ingress.yml `
  7. Run `kubectl -n yopass get ingress` to retrieve the IP. This may take some time to match with the **LoadBalancer** IP above. Point the domain name in your registrar to the IP address.
  8. Access the app using `https://your_domain_name`.

-----------------------------

Instructions to deploy **YoPass** on Azure Kubernetes Service using your own nginx ingress
  1. Deploy AKS cluster through Azure portal.
  2. Create a namespace. ` kubectl create ns yopass `
  3. Create a tls secret named ` cert-tls ` which has the domain's certificate & private key by running below command. The domain's .crt & .key file should already be present.

     ```
     kubectl -n yopass create secret tls cert-tls --cert=domain_name.crt --key=domain_name.key
     ```
  4. Deploy Nginx Ingress Controller by running below commands.

     ```
     helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
     helm repo update
     ```
     ```
     helm install nginx-ingress ingress-nginx/ingress-nginx \
     --set controller.service.externalTrafficPolicy=Local \
     --set controller.service.annotations."service\.beta\.kubernetes\.io/azure-load-balancer-health-probe-request-path"="/" \
     --set controller.service.enableHttps=true
     ```
  5. Run the command ` kubectl get svc nginx-ingress-ingress-nginx-controller ` to confirm if a **LoadBalancer** IP has been provisioned.

     ```
     pushkar [ ~ ]$ kubectl get svc nginx-ingress-ingress-nginx-controller
     NAME                                     TYPE           CLUSTER-IP    EXTERNAL-IP    PORT(S)                      AGE
     nginx-ingress-ingress-nginx-controller   LoadBalancer   10.0.58.180   20.174.45.64   80:32767/TCP,443:30598/TCP   110s
     ```  
  6. Deploy the `memcached` & `yopass` deployment & service using the `kubectl` command

     ```
     kubectl -n yopass apply -f yopass-dep.yml -f yopass-svc.yml -f memcached-dep.yml -f memcached-svc.yml
     ```
  7. Put the FQDN for which the secret has been created in ` nginx-ingress.yml ` file and then run the command ` kubectl -n yopass apply -f nginx-ingress.yml `
  8. Run `kubectl -n yopass get ingress` to retrieve the IP. This may take some time to match with the **LoadBalancer** IP above. Point the domain name in your registrar to the IP address.
  9. Access the app using `https://your_domain_name`.

-----------------------------

**Helm**
To install this app using Helm using the default **App Routing** add on, perform below steps
  1. Create a namespace. ` kubectl create ns yopass `
  2. Run the command ` kubectl -n app-routing-system get svc nginx ` to confirm if a **LoadBalancer** IP has been provisioned.

     ```
     pushkar [ ~ ]$ kubectl -n app-routing-system get svc nginx
     NAME    TYPE           CLUSTER-IP    EXTERNAL-IP    PORT(S)                      AGE
     nginx   LoadBalancer   10.0.58.180   20.174.45.64   80:32767/TCP,443:30598/TCP   110s
     ```
  3. Run the command to install **YoPass**

     ```
     helm install whisper ./helm --namespace yopass --set ingressClassName="web-app-routing" --set domain_name=your_preferred_fqdn --set-file tlsSecret.cert=domain_name.crt --set-file tlsSecret.key=domain_name.key
     ```
  4. Run `kubectl -n yopass get ingress` to retrieve the IP. This may take some time to match with the **LoadBalancer** IP above. Point the domain name in your registrar to the IP address.
  5. Access the app using `https://your_domain_name`.
  6. Uninstall the app using `helm uninstall whisper --namespace yopass`.

-----------------------------

**Helm**
To install this app using Helm using your own nginx ingress, perform below steps
  1. Create a namespace. ` kubectl create ns yopass `
  2. Deploy Nginx Ingress Controller by running below commands.
     
     ```
     helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
     helm repo update
     ```
     ```
     helm install nginx-ingress ingress-nginx/ingress-nginx \
     --set controller.service.externalTrafficPolicy=Local \
     --set controller.service.annotations."service\.beta\.kubernetes\.io/azure-load-balancer-health-probe-request-path"="/" \
     --set controller.service.enableHttps=true
     ```
  3. Run the command ` kubectl get svc nginx-ingress-ingress-nginx-controller ` to confirm if a **LoadBalancer** IP has been provisioned.

     ```
     pushkar [ ~ ]$ kubectl get svc nginx-ingress-ingress-nginx-controller
     NAME                                     TYPE           CLUSTER-IP    EXTERNAL-IP    PORT(S)                      AGE
     nginx-ingress-ingress-nginx-controller   LoadBalancer   10.0.58.180   20.174.45.64   80:32767/TCP,443:30598/TCP   110s
     ```
  4. Run the command to install **YoPass**

     ```
     helm install whisper ./helm --namespace yopass --set ingressClassName="nginx" --set domain_name=your_preferred_fqdn --set-file tlsSecret.cert=domain_name.crt --set-file tlsSecret.key=domain_name.key
     ```
  5. Run `kubectl -n yopass get ingress` to retrieve the IP. This may take some time to match with the **LoadBalancer** IP above. Point the domain name in your registrar to the IP address.
  6. Access the app using `https://your_domain_name`.
  7. Uninstall the app using `helm uninstall whisper --namespace yopass`.

-----------------------------

**ArgoCD** installation using the default **App Routing** add on

To install this app using ArgoCD, perform below steps
  1. Create a namespace. ` kubectl create ns argocd `.
  2. Create a tls secret named ` argocd-tls ` which has the domain's certificate & private key by running below command. The domain's .crt & .key file should already be present.

     ```
     kubectl -n argocd create secret tls argocd-tls --cert=argocd_domain_name.crt --key=argocd_domain_name.key
     ```
  3. Run the command ` kubectl -n app-routing-system get svc nginx ` to confirm if a **LoadBalancer** IP has been provisioned.

     ```
     pushkar [ ~ ]$ kubectl -n app-routing-system get svc nginx
     NAME    TYPE           CLUSTER-IP    EXTERNAL-IP    PORT(S)                      AGE
     nginx   LoadBalancer   10.0.58.180   20.174.45.64   80:32767/TCP,443:30598/TCP   110s
     ```
  4. Apply **ArgoCD** manifest file
     
     ```
     kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
     ```
  5. Patch the ConfigMap **argocd-cmd-params-cm** to add `server.insecure : true`, then scale the deployment to 0 & then 1

     ```
     kubectl -n argocd patch configmap argocd-cmd-params-cm --type merge -p '{"data":{"server.insecure":"true"}}'
     kubectl -n argocd scale deploy argocd-server --replicas=0
     kubectl -n argocd scale deploy argocd-server --replicas=1
     ```
  6. Put the FQDN for which the argocd secret has been created in ` argocd-web-app-routing-ingress.yml ` file and then run the command ` kubectl -n argocd apply -f argocd-web-app-routing-ingress.yml `
  7. Run ` kubectl -n argocd get ingress ` to retrieve the IP. This may take some time to match with the **LoadBalancer** IP above. Point the domain name in your registrar to the IP address.
  8. Access ArgoCD using ` https://argocd_domain_name `.
  9. To get the initial admin user password run the command

     ```
     kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 --decode && echo
     ```

-----------------------------

**ArgoCD** installation using **Nginx Ingress**

To install this app using ArgoCD, perform below steps
  1. Create a namespace. ` kubectl create ns argocd `.
  2. Create a tls secret named ` argocd-tls ` which has the domain's certificate & private key by running below command. The domain's .crt & .key file should already be present.

     ```
     kubectl -n argocd create secret tls argocd-tls --cert=argocd_domain_name.crt --key=argocd_domain_name.key
     ```
  3. Deploy Nginx Ingress Controller by running below commands.

     ```
     helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
     helm repo update
     ```
     ```
     helm install nginx-ingress ingress-nginx/ingress-nginx \
     --set controller.service.externalTrafficPolicy=Local \
     --set controller.service.annotations."service\.beta\.kubernetes\.io/azure-load-balancer-health-probe-request-path"="/" \
     --set controller.service.enableHttps=true
     ```
  4. Run the command ` kubectl get svc nginx-ingress-ingress-nginx-controller ` to confirm if a **LoadBalancer** IP has been provisioned.

     ```
     pushkar [ ~ ]$ kubectl get svc nginx-ingress-ingress-nginx-controller
     NAME                                     TYPE           CLUSTER-IP    EXTERNAL-IP    PORT(S)                      AGE
     nginx-ingress-ingress-nginx-controller   LoadBalancer   10.0.58.180   20.174.45.64   80:32767/TCP,443:30598/TCP   110s
     ```
  5. Apply **ArgoCD** manifest file
     
     ```
     kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
     ```
  6. Patch the ConfigMap **argocd-cmd-params-cm** to add `server.insecure : true`, then scale the deployment to 0 & then 1

     ```
     kubectl -n argocd patch configmap argocd-cmd-params-cm --type merge -p '{"data":{"server.insecure":"true"}}'
     kubectl -n argocd scale deploy argocd-server --replicas=0
     kubectl -n argocd scale deploy argocd-server --replicas=1
     ```
  7. Put the FQDN for which the argocd secret has been created in ` argocd-nginx-ingress.yml ` file and then run the command ` kubectl -n argocd apply -f argocd-nginx-ingress.yml `
  8. Run ` kubectl -n argocd get ingress ` to retrieve the IP. This may take some time to match with the **LoadBalancer** IP above. Point the domain name in your registrar to the IP address.
  9. Access ArgoCD using ` https://argocd_domain_name `.
 10. To get the initial admin user password run the command

     ```
     kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 --decode && echo
     ```

-----------------------------

**Prometheus** & **Grafana** monitoring using the default **App Routing** add on

To install the Prometheus stack, perform below steps
   1. Apply the monitoring stack manifest

      ```
      helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
      helm repo update
      helm install kube-prometheus prometheus-community/kube-prometheus-stack --create-namespace --namespace monitoring --set grafana.adminPassword='any_secure_password' --set grafana.service.type=ClusterIP
      ```

      ✅ This installs :- Prometheus, Grafana, Node Exporter, kube-state-metrics, Alertmanager
   2. Create a tls secret named ` monitoring-tls ` which has the Grafana domain's certificate & private key by running below command. The Grafana domain's .crt & .key file should already be present.

     ```
     kubectl -n monitoring create secret tls monitoring-tls --cert=grafana_domain_name.crt --key=grafana_domain_name.key
     ```
   3. Run the command ` kubectl -n app-routing-system get svc nginx ` to confirm if a **LoadBalancer** IP has been provisioned.

     ```
     pushkar [ ~ ]$ kubectl -n app-routing-system get svc nginx
     NAME    TYPE           CLUSTER-IP    EXTERNAL-IP    PORT(S)                      AGE
     nginx   LoadBalancer   10.0.58.180   20.174.45.64   80:32767/TCP,443:30598/TCP   110s
     ```
   4. Put the FQDN for which the monitoring secret has been created in ` monitoring-web-app-routing-ingress.yml ` file and then run the command ` kubectl -n monitoring apply -f monitoring-web-app-routing-ingress.yml `

      ```
      kubectl -n monitoring apply -f monitoring-web-app-routing-ingress.yml
      ```
   5. Run ` kubectl -n monitoring get ingress ` to retrieve the IP. This may take some time to match with the **LoadBalancer** IP above. Point the domain name in your registrar to the IP address.
   6. Access Grafana using ` https://grafana_domain_name `.
   7. To get the initial admin user password run the command

      ```
      kubectl -n monitoring get secrets kube-prometheus-grafana -o jsonpath="{.data.admin-password}" | base64 -d ; echo
      ```

-----------------------------

**Prometheus** & **Grafana** monitoring using **Nginx Ingress**

To install the Prometheus stack, perform below steps
   1. Apply the monitoring stack manifest

      ```
      helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
      helm repo update
      helm install kube-prometheus prometheus-community/kube-prometheus-stack --create-namespace --namespace monitoring --set grafana.adminPassword='any_secure_password' --set grafana.service.type=ClusterIP
      ```

      ✅ This installs :- Prometheus, Grafana, Node Exporter, kube-state-metrics, Alertmanager
   2. Create a tls secret named ` monitoring-tls ` which has the Grafana domain's certificate & private key by running below command. The Grafana domain's .crt & .key file should already be present.

     ```
     kubectl -n monitoring create secret tls monitoring-tls --cert=grafana_domain_name.crt --key=grafana_domain_name.key
     ```
   3. Deploy Nginx Ingress Controller by running below commands.

     ```
     helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
     helm repo update
     ```
     ```
     helm install nginx-ingress ingress-nginx/ingress-nginx \
     --set controller.service.externalTrafficPolicy=Local \
     --set controller.service.annotations."service\.beta\.kubernetes\.io/azure-load-balancer-health-probe-request-path"="/" \
     --set controller.service.enableHttps=true
     ```
  4. Run the command ` kubectl get svc nginx-ingress-ingress-nginx-controller ` to confirm if a **LoadBalancer** IP has been provisioned.

     ```
     pushkar [ ~ ]$ kubectl get svc nginx-ingress-ingress-nginx-controller
     NAME                                     TYPE           CLUSTER-IP    EXTERNAL-IP    PORT(S)                      AGE
     nginx-ingress-ingress-nginx-controller   LoadBalancer   10.0.58.180   20.174.45.64   80:32767/TCP,443:30598/TCP   110s
     ```
  5. Put the FQDN for which the monitoring secret has been created in ` monitoring-nginx-ingress.yml ` file and then run the command ` kubectl -n monitoring apply -f monitoring-nginx-ingress.yml `

     ```
     kubectl -n monitoring apply -f monitoring-nginx-ingress.yml
     ```
  6. Run ` kubectl -n monitoring get ingress ` to retrieve the IP. This may take some time to match with the **LoadBalancer** IP above. Point the domain name in your registrar to the IP address.
  7. Access Grafana using ` https://grafana_domain_name `.
  8. To get the initial admin user password run the command

     ```
     kubectl -n monitoring get secrets kube-prometheus-grafana -o jsonpath="{.data.admin-password}" | base64 -d ; echo
     ```