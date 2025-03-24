Instructions to deploy **YoPass** on AKS
  1. Deploy AKS cluster through Azure portal.
  2. Deploy Nginx Ingress Controller by running below commands

     ` helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx `
     ` helm repo update `
     ` helm install nginx-ingress ingress-nginx/ingress-nginx --set controller.service.externalTrafficPolicy=Local --set controller.service.annotations."service\.beta\.kubernetes\.io/azure-load-balancer-health-probe-request-path"="/" --set controller.service.enableHttps=true `
  4. Create a namespace. ` kubectl create ns yopass `
  5. Deploy the `memcached` & `yopass` deployment & service using the `kubectl` command

     ` kubectl -n yopass apply -f yopass-dep.yml -f yopass-svc.yml -f memcached-dep.yml -f memcached-svc.yml `
  6. Deploy `Ingress`, `Ingress Class` & `Ingress Params` which will create an ALB listening on port 443. We will need to provide the arn of the certificate in the `Ingress` object. The certificate for the domain name needs to be created in ACM and DNS validation or Email validation needs to be done prior to creating these resources.
  7. Run the command ` kubectl -n yopass apply -f ingress-all.yml `
  8. Run `kubectl -n yopass get ingress` to retrieve the ALB DNS.
  9. Point the domain name in Route 53 to the ALB as an A (alias) record.
  10. Access the app using `https://your_domain_name`.

-----------------------------

**Helm**
To install this app using Helm, perform below steps
  1. Generate a certificate from ACM for your domain name. The certificate arn will be required in the next step since we are running ALB on port 443.
  2. Run the command

     `helm install whisper ./helm --namespace yopass --create-namespace --set certificate_arn=arn_got_from_previous_step`.
  3. Get the ALB DNS using `kubectl -n yopass get ingress` and point the domain name in Route 53 to the ALB as an A (alias) record.
  4. Access the app using `https://your_domain_name`.
  5. Uninstall the app using `helm unistall whisper --namespace whisper`.
