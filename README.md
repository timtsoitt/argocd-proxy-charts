# argocd-proxy-charts

## 1. Install Argo CD and Sealed Secrets related resources
```
helm repo add argo https://argoproj.github.io/argo-helm
helm install my-argo-cd argo/argo-cd --version 4.8.3

helm repo add bitnami-labs https://bitnami-labs.github.io/sealed-secrets/
helm install sealed-secrets-controller bitnami-labs/sealed-secrets --version 2.2.0

# If you are not using MacOS, you can go to Sealed Secrets repository to download kubeseal binary.
# https://github.com/bitnami-labs/sealed-secrets
brew install kubeseal
```

## (Optional) 2. Generate SealedSecret
You can try to use kubeseal to encrypt secret.

`kubeseal --controller-namespace default --format yaml -f charts/nginx/manifests/mysecret.yaml > charts/nginx/templates/mysealedsecret.yaml`

## 3. Access Argo CD portal
```
# Get initial admin password
kubectl get secrets argocd-initial-admin-secret -o jsonpath='{.data.password}' | base64 -d

# Port forward Argo CD server
kubectl port-forward service/my-argo-cd-argocd-server -n default 8080:443
```

Access https://localhost:8080/

username: admin\
password: (Refer to the output)


## 4. Install Argo CD application
Run this command, and then Argo CD will auto sync your application.

`kubectl apply -f nginx-application.yaml`

## 5. Verify the result
If everything works well, you will see the secret is deployed to the cluster.

```
kubectl get secrets my-secret
kubectl get secret/my-secret --template={{.data.secret}} | base64 -D    
```

## 6. Cleanup

```
kubectl delete -f nginx-application.yaml
helm uninstall my-argo-cd 
helm uninstall sealed-secrets-controller
```