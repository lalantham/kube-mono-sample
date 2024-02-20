### Install Kyverno using Helm
```bash
helm repo add kyverno https://kyverno.github.io/kyverno/
helm repo update
helm install kyverno kyverno/kyverno -n kyverno --create-namespace
```

### Create Kubernetes ImagePull Secret
```bash
kubectl create secret docker-registry ocirsecret --docker-server='sin.ocir.io' --docker-username='bmkzpkoomgkt/oracleidentitycloudservice/lalantha.madhushan@lexiicontech.com' --docker-password=')by:toP{YWZ9M1gin-xs'
```
### Define required Kyverno ClusterPolicies
```bash
nano add-imagepullsecret.yaml
```
```bash
---
apiVersion: kyverno.io/v1
kind: ClusterPolicy
metadata:
  name: inject-imagepullsecret-to-namespace
  annotations:
    policies.kyverno.io/title: Clone imagePullSecret secret to new namespaces
    policies.kyverno.io/subject: Namespace
    policies.kyverno.io/description: >-
      ImagePullSecrets must be present in the same namespace as the pods using them.
      This policy monitors for new namespaces being created (except kube-system and kyverno),
      and automatically clones into the namespace the `ocirsecret` from the `default` namespace.
spec:
  generateExisting: true
  rules:
  - name: inject-imagepullsecret-to-namespace
    match:
      any:
      - resources:
          kinds:
          - Namespace
    exclude:
      any:
      - resources:
          namespaces:
          - kube-system
          - kube-node-lease
          - kube-public
          - kyverno
    generate:
      apiVersion: v1
      kind: Secret
      name: ocirsecret
      namespace: "{{ request.object.metadata.name }}"
      synchronize: true
      clone:
        namespace: default
        name: ocirsecret
---
apiVersion: kyverno.io/v1
kind: ClusterPolicy
metadata:
  name: add-imagepullsecrets
  annotations:
    policies.kyverno.io/title: Add imagePullSecrets
    policies.kyverno.io/subject: Pod
    policies.kyverno.io/description: >-
      Images coming from certain registries require authentication in order to pull them,
      and the kubelet uses this information in the form of an imagePullSecret to pull
      those images on behalf of your Pod. This policy searches pod spec for images coming from a
      registry which contains `sin.ocir.io/bmkzpkoomgkt` and, if found, will mutate the Pod
      to add an imagePullSecret called `ocirsecret`.
spec:
  rules:
  - name: add-imagepullsecret
    match:
      any:
      - resources:
          kinds:
          - Pod
    context:
    - name: images_in_ocir
      variable:
        jmesPath: "[request.object.spec.containers[*].image.contains(@, 'sin.ocir.io/bmkzpkoomgkt'), request.object.spec.initContainers[*].image.contains(@, 'sin.ocir.io/bmkzpkoomgkt')][]"
        default: []
    preconditions:
      all:
      - key: true
        operator: In
        value: "{{ images_in_ocir }}"
    mutate:
      patchStrategicMerge:
        spec:
          imagePullSecrets:
          - name: ocirsecret
```
Replace "sin.ocir.io/bmkzpkoomgkt" with working namespace.
```bash
kubectl apply -f add-imagepullsecret.yaml
```

Reference: https://docs.oracle.com/en/learn/oke-kyverno-imagepullsecrets/index.html
