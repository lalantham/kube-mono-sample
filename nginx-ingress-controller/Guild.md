### Add Helm Repo
```bash
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
helm repo update
```

### Install Nginx Ingress Controller
```bash
helm install nginx-ingress ingress-nginx/ingress-nginx --set controller.publishService.enabled=true
```

### Check Load Balancer Availability
```bash
kubectl --namespace default get services -o wide -w nginx-ingress-ingress-nginx-controller
```

### Sample Ingress File
nano kubernetes-ingress.yaml
```bash
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: hello-kubernetes-ingress
  annotations:
    kubernetes.io/ingress.class: nginx
spec:
  rules:
  - host: "hw1.your_domain_name"
    http:
      paths:
      - pathType: Prefix
        path: "/"
        backend:
          service:
            name: hello-kubernetes-first
            port:
              number: 80
  - host: "hw2.your_domain_name"
    http:
      paths:
      - pathType: Prefix
        path: "/"
        backend:
          service:
            name: hello-kubernetes-second
            port:
              number: 80
```
