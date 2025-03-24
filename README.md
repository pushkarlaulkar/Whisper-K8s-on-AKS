Instructions to deploy **YoPass** on Azure Kubernetes Service
  1. Deploy AKS cluster through Azure portal.
  2. Deploy Nginx Ingress Controller by running below commands

     ` helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx `
     
     ` helm repo update `
     
     ``` helm install nginx-ingress ingress-nginx/ingress-nginx --set controller.service.externalTrafficPolicy=Local --set controller.service.annotations."service\.beta\.kubernetes\.io/azure-load-balancer-health-probe-request-path"="/" --set controller.service.enableHttps=true ```
  3. Create a namespace. ` kubectl create ns yopass `
  4. Deploy the `memcached` & `yopass` deployment & service using the `kubectl` command

     ` kubectl -n yopass apply -f yopass-dep.yml -f yopass-svc.yml -f memcached-dep.yml -f memcached-svc.yml `
  5. Create a tls secret named ` cert-tls ` which has the domain's certificate & private key by running below command. The domain's .crt & .key file should be already be present.

     ` kubectl -n yopass create tls cert-tls --cert=domain_name.crt --key=domain_name.key `
  6. Put the domain name for which the secret has been created in ` ingress.yml ` file and then run the command ` kubectl -n yopass apply -f ingress.yml `
  7. Run `kubectl -n yopass get ingress` to retrieve the IP.
  8. Point the domain name in your registrar to the IP address.
  9. Access the app using `https://your_domain_name`.

-----------------------------

**Helm**
To install this app using Helm, perform below steps
  1. Generate a certificate from ACM for your domain name. The certificate arn will be required in the next step since we are running ALB on port 443.
  2. Run the command

     `helm install whisper ./helm --namespace yopass --create-namespace --set certificate_arn=arn_got_from_previous_step`.
  3. Get the ALB DNS using `kubectl -n yopass get ingress` and point the domain name in Route 53 to the ALB as an A (alias) record.
  4. Access the app using `https://your_domain_name`.
  5. Uninstall the app using `helm unistall whisper --namespace whisper`.
