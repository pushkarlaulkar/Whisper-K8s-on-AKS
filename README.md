Instructions to deploy **YoPass** on Azure Kubernetes Service using the default **App Routing** add on
  1. Deploy AKS cluster through Azure portal.
  2. Create a namespace. ` kubectl create ns yopass `
  3. Create a tls secret named ` cert-tls ` which has the domain's certificate & private key by running below command. The domain's .crt & .key file should already be present.

     ```
     kubectl -n yopass create secret tls cert-tls --cert=domain_name.crt --key=domain_name.key
     ```
  4. Run the command ` kubectl -n app-routing get svc nginx ` to confirm if a **LoadBalancer** IP has been provisioned.

     ```
     pushkar [ ~ ]$ kubectl -n app-routing get svc nginx
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
  4. Deploy Nginx Ingress Controller by running below commands. During install the tls secret created above will be specified to be used as default ssl certificate.

     ```
     helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
     helm repo update
     ```
     ```
     helm install nginx-ingress ingress-nginx/ingress-nginx \
     --set controller.service.externalTrafficPolicy=Local \
     --set controller.service.annotations."service\.beta\.kubernetes\.io/azure-load-balancer-health-probe-request-path"="/" \
     --set controller.service.enableHttps=true \
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
  7. Put the FQDN for which the secret has been created in ` nginx-ingress.yml ` file and then run the command ` kubectl -n yopass apply -f ingress.yml `
  8. Run `kubectl -n yopass get ingress` to retrieve the IP. This may take some time to match with the **LoadBalancer** IP above. Point the domain name in your registrar to the IP address.
  9. Access the app using `https://your_domain_name`.

-----------------------------

**Helm**
To install this app using Helm, perform below steps
  1. Create a namespace. ` kubectl create ns yopass `
  2. Create a tls secret named ` cert-tls ` which has the domain's certificate & private key by running below command. The domain's .crt & .key file should already be present.

     ```
     kubectl -n yopass create secret tls cert-tls --cert=domain_name.crt --key=domain_name.key
     ```
  3. Deploy Nginx Ingress Controller by running below commands. During install the tls secret created above will be specified to be used as default ssl certificate.
     
     ```
     helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
     helm repo update
     ```
     ```
     helm install nginx-ingress ingress-nginx/ingress-nginx \
     --set controller.service.externalTrafficPolicy=Local \
     --set controller.service.annotations."service\.beta\.kubernetes\.io/azure-load-balancer-health-probe-request-path"="/" \
     --set controller.service.enableHttps=true \
     --set controller.extraArgs.default-ssl-certificate=yopass/cert-tls
     ```
  4. Run the command ` kubectl get svc nginx-ingress-ingress-nginx-controller ` to confirm if a **LoadBalancer** IP has been provisioned.

     ```
     pushkar [ ~ ]$ kubectl get svc nginx-ingress-ingress-nginx-controller
     NAME                                     TYPE           CLUSTER-IP    EXTERNAL-IP    PORT(S)                      AGE
     nginx-ingress-ingress-nginx-controller   LoadBalancer   10.0.58.180   20.174.45.64   80:32767/TCP,443:30598/TCP   110s
     ```
  5. Run the command to install **YoPass**

     ```
     helm install whisper ./helm --namespace yopass --set domain_name=your_preferred_fqdn
     ```
  6. Run `kubectl -n yopass get ingress` to retrieve the IP. This may take some time to match with the **LoadBalancer** IP above. Point the domain name in your registrar to the IP address.
  7. Access the app using `https://your_domain_name`.
  8. Uninstall the app using `helm uninstall whisper --namespace yopass`.
