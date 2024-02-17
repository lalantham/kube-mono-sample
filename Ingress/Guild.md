### Install Helm
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
helm repo update
helm install <release-name (my-ingress)> ingress-nginx/ingress-nginx -n <namespace>
