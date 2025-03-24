Instructions to deploy **YoPass** on Azure Kubernetes Service
  1. Deploy AKS cluster through Azure portal.
  2. Deploy Nginx Ingress Controller by running below commands

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
  4. Create a namespace. ` kubectl create ns yopass `
  5. Deploy the `memcached` & `yopass` deployment & service using the `kubectl` command

     ```
     kubectl -n yopass apply -f yopass-dep.yml -f yopass-svc.yml -f memcached-dep.yml -f memcached-svc.yml
     ```
  7. Create a tls secret named ` cert-tls ` which has the domain's certificate & private key by running below command. The domain's .crt & .key file should already be present.

     ```
     kubectl -n yopass create secret tls cert-tls --cert=domain_name.crt --key=domain_name.key
     ```
  9. Put the FQDN for which the secret has been created in ` ingress.yml ` file and then run the command ` kubectl -n yopass apply -f ingress.yml `
  10. Run `kubectl -n yopass get ingress` to retrieve the IP. Point the domain name in your registrar to the IP address.
  11. Access the app using `https://your_domain_name`.

-----------------------------

**Helm**
To install this app using Helm, perform below steps
  1. Deploy Nginx Ingress Controller by running below commands

     ` helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx `
     
     ` helm repo update `
     ```
     helm install nginx-ingress ingress-nginx/ingress-nginx \
     --set controller.service.externalTrafficPolicy=Local \
     --set controller.service.annotations."service\.beta\.kubernetes\.io/azure-load-balancer-health-probe-request-path"="/" \
     --set controller.service.enableHttps=true
     ```
  3. Run the command

     ```
     helm install whisper ./helm --namespace yopass --create-namespace \
     --set tls.cert="$(cat domain_name.crt | base64 -w 0)" \
     --set tls.key="$(cat domain_name.key | base64 -w 0) \
     --set domain_name=your_preferred_fqdn
     ```
  5. Run `kubectl -n yopass get ingress` to retrieve the IP. Point the domain name in your registrar to the IP address.
  6. Access the app using `https://your_domain_name`.
  7. Uninstall the app using `helm uninstall whisper --namespace yopass`.
