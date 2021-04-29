## 1. Add to Appmesh on AWS Console

AWS Console -> AppMesh -> Create mesh
- Mesh name: `sample-appmesh`
- Egress filter: `Allow external traffic`

## 2. Add to Custom Resource Definitions

Reference to notion

## 3. Add to Appmesh Controller

Reference to aws-cdk on gitHub

## 4. Add to namespace.yaml

```
kubectl apply -f namespace.yaml
```