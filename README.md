## 1. Add to CloudMap on AWS Console

AWS Console -> CloudMap -> Create namespace
- Namespace name: `skill53.local`
- Namespace description: `This is discovery for sample-appmesh`
- Instance discovery: `API calls and DNS queries in VPCs`

## 2. Add to Custom Resource Definitions

```
kubens app
kubectl apply -k "github.com/aws/eks-charts/stable/appmesh-controller//crds?ref=master"
```

## 3. Add to Appmesh Controller

```
curl -o controller-iam-policy.json https://raw.githubusercontent.com/aws/aws-app-mesh-controller-for-k8s/master/config/iam/controller-iam-policy.json
```
```
aws iam create-policy \
    --policy-name AWSAppMeshK8sControllerIAMPolicy \
    --policy-document file://controller-iam-policy.json
```
```
eksctl utils associate-iam-oidc-provider --region=$AWS_REGION \
    --cluster=$CLUSTER_NAME \
    --approve
```
```
eksctl create iamserviceaccount --cluster $CLUSTER_NAME \
    --namespace appmesh-system \
    --name appmesh-controller \
    --attach-policy-arn arn:aws:iam::$AWS_ACCOUNT_ID:policy/AWSAppMeshK8sControllerIAMPolicy  \
    --override-existing-serviceaccounts \
    --approve
```
```
helm repo add eks https://aws.github.io/eks-charts
helm upgrade -i appmesh-controller eks/appmesh-controller \
    --namespace appmesh-system \
    --set region=$AWS_REGION \
    --set serviceAccount.create=false \
    --set serviceAccount.name=appmesh-controller
```

## 4. Add to mesh.yaml

```
kubectl apply -f mesh.yaml
```

## 5. Add to namespace.yaml

```
kubectl apply -f namespace.yaml
```

## 6. Add to IRSA for EnvoyAccess and X-Ray write

https://docs.aws.amazon.com/app-mesh/latest/userguide/proxy-authorization.html

Managed policy
- `AWSAppMeshEnvoyAccess`
- `AWSXrayWriteOnlyAccess`
