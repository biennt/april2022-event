# april2022-event
##This is a demo for nginx ingress controller with NAP (Nginx AppProtect)

### Install K8S on Ubuntu 20.04
Follow this guide to have container engine, kubectl, kubelet, kubeadm on your k8s nodes
https://computingforgeeks.com/deploy-kubernetes-cluster-on-ubuntu-with-kubeadm/

Init the cluster:
```
sudo kubeadm init --config kubeadm.conf
```
Get the config file ready for kubectl and/or other k8s client such as k9s, lens..
```
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```
Join other worker nodes by the command provided by the above kubeadm init, example:
```
sudo kubeadm join 10.0.0.191:6443 --token rqm836.wmiw7pw2pm0mi5ul --discovery-token-ca-cert-hash sha256:00cff16199ea4c9bbed562499fe41e81d0c5346206be39938624fdcffb768476
```
Install network interface:
```
kubectl apply -f "https://cloud.weave.works/k8s/net?k8s-version=$(kubectl version | base64 | tr -d '\n')"
```
### Building the container image for nginx plus
(and optionally, with Nginx AppProtect and AppProtect DOS)
```
git clone https://github.com/nginxinc/kubernetes-ingress.git --branch v2.2.0
cp nginx-repo.* kubernetes-ingress/
cd kubernetes-ingress
make debian-image-nap-dos-plus PREFIX=your.private.registry/nginx-plus-ingress TARGET=download DOCKER_BUILD_OPTIONS="--pull --no-cache" 
make push PREFIX=your.private.registry/nginx-plus-ingress
```
### Installation of nginx ingress controller on your k8s
you need to install/apply some other configs such as namespace, service account, rbac, default secret, crds..
```
cd deployments
kubectl apply -f common/ns-and-sa.yaml
kubectl apply -f rbac/rbac.yaml
kubectl apply -f rbac/ap-rbac.yaml
kubectl apply -f rbac/apdos-rbac.yaml
kubectl apply -f common/default-server-secret.yaml
kubectl apply -f common/nginx-config.yaml
kubectl apply -f common/ingress-class.yaml
kubectl apply -f common/crds/k8s.nginx.org_virtualservers.yaml
kubectl apply -f common/crds/k8s.nginx.org_virtualserverroutes.yaml
kubectl apply -f common/crds/k8s.nginx.org_transportservers.yaml
kubectl apply -f common/crds/k8s.nginx.org_policies.yaml
kubectl apply -f common/crds/k8s.nginx.org_globalconfigurations.yaml
kubectl apply -f common/crds/appprotect.f5.com_aplogconfs.yaml
kubectl apply -f common/crds/appprotect.f5.com_appolicies.yaml
kubectl apply -f common/crds/appprotect.f5.com_apusersigs.yaml
kubectl apply -f common/crds/appprotectdos.f5.com_apdoslogconfs.yaml
kubectl apply -f common/crds/appprotectdos.f5.com_apdospolicy.yaml
kubectl apply -f common/crds/appprotectdos.f5.com_dosprotectedresources.yaml
kubectl apply -f deployment/appprotect-dos-arb.yaml
kubectl apply -f service/appprotect-dos-arb-svc.yaml
```
If your private registry requires authentication, you need to create a secret so k8s know how to authenticate while it downloads the image:
```
kubectl create secret docker-registry regcred --docker-server=gitlab.bienlab.com:5005 --docker-username=bien --docker-password='abc123$'
```
Edit the deployment file (in this case, i use daemonsets, so the file is daemon-set/nginx-plus-ingress.yaml)
```
imagePullSecrets:
- name: regcred
```
change the image repo in this file
```
- image: your.private.registry/nginx-plus-ingress:<tag>
```
You may want to enable AppProtect and AppProtect DOS
```
-enable-app-protect
-enable-app-protect-dos
```

Finally, apply the yaml:
```
kubectl apply -f daemon-set/nginx-plus-ingress.yaml
```
  
### Run the examples
Once you have the ingress controller installed, you can go with examples from here: https://github.com/nginxinc/kubernetes-ingress/tree/v2.2.0/examples
  - basic: https://github.com/nginxinc/kubernetes-ingress/tree/v2.2.0/examples/complete-example 
  - with AppProtect: https://github.com/nginxinc/kubernetes-ingress/tree/v2.2.0/examples/appprotect
  
 Enjoy!
 
