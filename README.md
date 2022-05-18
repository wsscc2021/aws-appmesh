## 1. CloudMap on AWS Console

AWS Console -> CloudMap -> Create namespace
- Namespace name: `skill53.local`
- Namespace description: `This is discovery for sample-appmesh`
- Instance discovery: `API calls and DNS queries in VPCs`

## 2. Custom Resource Definitions

```
kubectl apply -k "github.com/aws/eks-charts/stable/appmesh-controller//crds?ref=master" -n ${app_namespace}
```

## 3. appmesh-controller

- associate oidc provider
    ```
    eksctl utils associate-iam-oidc-provider --region=$AWS_REGION \
        --cluster=$CLUSTER_NAME \
        --approve
    ```

- create iam policy for appmesh-controller
    ```
    curl -o controller-iam-policy.json https://raw.githubusercontent.com/aws/aws-app-mesh-controller-for-k8s/master/config/iam/controller-iam-policy.json
    ```
    ```
    aws iam create-policy \
        --policy-name AWSAppMeshK8sControllerIAMPolicy \
        --policy-document file://controller-iam-policy.json
    ```

- create iam role for service account 
    ```
    eksctl create iamserviceaccount --cluster $CLUSTER_NAME \
        --namespace appmesh-system \
        --name appmesh-controller \
        --attach-policy-arn arn:aws:iam::$AWS_ACCOUNT_ID:policy/AWSAppMeshK8sControllerIAMPolicy  \
        --override-existing-serviceaccounts \
        --approve
    ```

- add helm repo
    ```
    helm repo add eks https://aws.github.io/eks-charts
    ```

- install helm chart
    ```
    helm upgrade -i appmesh-controller eks/appmesh-controller \
        --namespace appmesh-system \
        --set region=$AWS_REGION \
        --set serviceAccount.create=false \
        --set serviceAccount.name=appmesh-controller
    ```
    
    - if you need tolerations.
        ```
        helm repo add eks https://aws.github.io/eks-charts
        helm upgrade -i appmesh-controller eks/appmesh-controller \
        --namespace appmesh-system \
        --set region=$AWS_REGION \
        --set serviceAccount.create=false \
        --set serviceAccount.name=appmesh-controller \
        --set tolerations\[0\].key="Management" \
        --set tolerations\[0\].value="Tools" \
        --set tolerations\[0\].effect="NoSchedule"
        ```

## 4. appmesh resources

- appmesh - mesh
    ```
    kubectl apply -f mesh.yaml
    ```

- add annotation to namespace
    ```
    kubectl apply -f namespace.yaml
    ```

- iam role for service account to virtual node
    ```
    eksctl create iamserviceaccount --cluster $CLUSTER_NAME \
        --namespace ${app_namespace} \
        --name appmesh-controller \
        --attach-policy-arn arn:aws:iam::aws:policy/AWSAppMeshEnvoyAccess \
        --override-existing-serviceaccounts \
        --approve
    ```
    - https://docs.aws.amazon.com/app-mesh/latest/userguide/proxy-authorization.html
    - `AWSAppMeshEnvoyAccess`
    - `AWSXrayWriteOnlyAccess`

## 5. allow external traffic

- virtual-gateway - envoy proxy healthcheck
    - tcp port `9901`
    - http path `/server_info`