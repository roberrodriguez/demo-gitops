# Demo ArgoCD

## 1. Creacion cluster kubernetes con Kind (Kubernetes in docker)
**kind create cluster --config kind-config.yaml**
```bash
~/demo-gitops git:(main|)❯❯❯ kind create cluster --config kind-config.yaml
Creating cluster "kind" ...
 ✓ Ensuring node image (kindest/node:v1.31.0) 🖼
 ✓ Preparing nodes 📦 📦
 ✓ Writing configuration 📜
 ✓ Starting control-plane 🕹️
 ✓ Installing CNI 🔌
 ✓ Installing StorageClass 💾
 ✓ Joining worker nodes 🚜
Set kubectl context to "kind-kind"
You can now use your cluster with:

kubectl cluster-info --context kind-kind

Have a nice day! 👋
~/demo-gitops git:(main|)❯❯❯
```

## 2. Instalación ArgoCD

### Creamos el namespace
**kubectl create ns argocd**
```bash
~/demo-gitops git:(main|)❯❯❯ kubectl create ns argocd
namespace/argocd created
~/demo-gitops git:(main|)❯❯❯
```

### Instalamos ArgoCD
**kubectl -n argocd apply -f https://raw.githubusercontent.com/argoproj/argo-cd/v2.12.2/manifests/install.yaml**
```bash
~/demo-gitops git:(main|)❯❯❯ kubectl -n argocd apply -f https://raw.githubusercontent.com/argoproj/argo-cd/v2.12.2/manifests/install.yaml
customresourcedefinition.apiextensions.k8s.io/applications.argoproj.io created
customresourcedefinition.apiextensions.k8s.io/applicationsets.argoproj.io created
customresourcedefinition.apiextensions.k8s.io/appprojects.argoproj.io created
serviceaccount/argocd-application-controller created
serviceaccount/argocd-applicationset-controller created
serviceaccount/argocd-dex-server created
serviceaccount/argocd-notifications-controller created
serviceaccount/argocd-redis created
serviceaccount/argocd-repo-server created
serviceaccount/argocd-server created
role.rbac.authorization.k8s.io/argocd-application-controller created
role.rbac.authorization.k8s.io/argocd-applicationset-controller created
role.rbac.authorization.k8s.io/argocd-dex-server created
role.rbac.authorization.k8s.io/argocd-notifications-controller created
role.rbac.authorization.k8s.io/argocd-redis created
role.rbac.authorization.k8s.io/argocd-server created
clusterrole.rbac.authorization.k8s.io/argocd-application-controller created
clusterrole.rbac.authorization.k8s.io/argocd-applicationset-controller created
clusterrole.rbac.authorization.k8s.io/argocd-server created
rolebinding.rbac.authorization.k8s.io/argocd-application-controller created
rolebinding.rbac.authorization.k8s.io/argocd-applicationset-controller created
rolebinding.rbac.authorization.k8s.io/argocd-dex-server created
rolebinding.rbac.authorization.k8s.io/argocd-notifications-controller created
rolebinding.rbac.authorization.k8s.io/argocd-redis created
rolebinding.rbac.authorization.k8s.io/argocd-server created
clusterrolebinding.rbac.authorization.k8s.io/argocd-application-controller created
clusterrolebinding.rbac.authorization.k8s.io/argocd-applicationset-controller created
clusterrolebinding.rbac.authorization.k8s.io/argocd-server created
configmap/argocd-cm created
configmap/argocd-cmd-params-cm created
configmap/argocd-gpg-keys-cm created
configmap/argocd-notifications-cm created
configmap/argocd-rbac-cm created
configmap/argocd-ssh-known-hosts-cm created
configmap/argocd-tls-certs-cm created
secret/argocd-notifications-secret created
secret/argocd-secret created
service/argocd-applicationset-controller created
service/argocd-dex-server created
service/argocd-metrics created
service/argocd-notifications-controller-metrics created
service/argocd-redis created
service/argocd-repo-server created
service/argocd-server created
service/argocd-server-metrics created
deployment.apps/argocd-applicationset-controller created
deployment.apps/argocd-dex-server created
deployment.apps/argocd-notifications-controller created
deployment.apps/argocd-redis created
deployment.apps/argocd-repo-server created
deployment.apps/argocd-server created
statefulset.apps/argocd-application-controller created
networkpolicy.networking.k8s.io/argocd-application-controller-network-policy created
networkpolicy.networking.k8s.io/argocd-applicationset-controller-network-policy created
networkpolicy.networking.k8s.io/argocd-dex-server-network-policy created
networkpolicy.networking.k8s.io/argocd-notifications-controller-network-policy created
networkpolicy.networking.k8s.io/argocd-redis-network-policy created
networkpolicy.networking.k8s.io/argocd-repo-server-network-policy created
networkpolicy.networking.k8s.io/argocd-server-network-policy created
~/demo-gitops git:(main|)❯❯❯
```

### Parcheamos ArgoCD para que use el proxy de T-Systems
**kubectl -n argocd patch deployment argocd-repo-server -p '{"spec":{"template":{"spec":{"containers":[{"name":"argocd-repo-server","env":[{"name":"HTTP_PROXY","value":"http://10.49.1.1:8080"},{"name":"HTTPS_PROXY","value":"http://10.49.1.1:8080"},{"name":"NO_PROXY","value":".cluster.local,.svc,10.244.0.0/16,172.18.0.0/16"}]}]}}}}'**
```bash
~/demo-gitops git:(main|)❯❯❯ kubectl -n argocd patch deployment argocd-repo-server -p '{"spec":{"template":{"spec":{"containers":[{"name":"argocd-repo-server","env":[{"name":"HTTP_PROXY","value":"http://10.49.1.1:8080"},{"name":"HTTPS_PROXY","value":"http://10.49.1.1:8080"},{"name":"NO_PROXY","value":".cluster.local,.svc,10.244.0.0/16,172.18.0.0/16"}]}]}}}}'
deployment.apps/argocd-repo-server patched
~/demo-gitops git:(main|)❯❯❯ 
```


### Creamos el Servicio, Proyecto y Aplicaciones
**kubectl -n argocd apply -f argo**
```bash
~/demo-gitops git:(main|)❯❯❯ kubectl -n argocd apply -f argo
applicationset.argoproj.io/sampleapp created
service/argo-svc created
appproject.argoproj.io/sampleapp-prj created
~/demo-gitops git:(main|…1)❯❯❯ 
```

## Servicios desplegados
### ArgoCD
https://localhost:30080

### Sample APP - DEV
http://localhost:30001

### Sample App - PROD
http://localhost:30000

### Para obtener el password de admin de ArgoCD
**kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d; echo**
```bash
~/demo-gitops git:(main|…1)❯❯❯ kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d; echo
qUfoHqZYY86v97Q5
~/demo-gitops git:(main|…1)❯❯❯
```
