### File Order
```bash
mongo-secret.yaml
mongo.yaml
mongo-configmap.yaml
mongo-express.yaml
```

### OCI Internal Load Balancer Service
```bash
apiVersion: v1
kind: Service
metadata:
  name: mongo-express-service
  labels:
    app: mongo-express
  annotations:
    oci.oraclecloud.com/load-balancer-type: "lb"
    service.beta.kubernetes.io/oci-load-balancer-internal: "true"
    service.beta.kubernetes.io/oci-load-balancer-subnet1: "ocid1.subnet.oc1.ap-singapore-1.aaaaaaaa5vr6y4zfz77bbn7lbrb3f6pkm63knhd4jxbajbh3oklpa6tqsbba"
spec:
  type: LoadBalancer
  ports:
  - port: 8081
  selector:
    app: mongo-express
