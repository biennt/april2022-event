# april2022-event
##This is a demo for nginx ingress controller with NAP (Nginx AppProtect)

### Install K8S on Ubuntu 20.04
Follow this guide to have containerd, kubectl, kubelet, kubeadm on your k8s nodes
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
```
rm -rf kubernetes-ingress
git clone https://github.com/nginxinc/kubernetes-ingress.git --branch v2.2.0
cp nginx-repo.* kubernetes-ingress/
cd kubernetes-ingress

#build the container image
make debian-image-nap-dos-plus PREFIX=gitlab.bienlab.com:5005/bien/kic/nginx-plus-ingress TARGET=download DOCKER_BUILD_OPTIONS="--pull --no-cache" 
make push PREFIX=gitlab.bienlab.com:5005/bien/kic/nginx-plus-ingress


#### configs
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


# create image pull secret
kubectl create secret docker-registry regcred --docker-server=gitlab.bienlab.com:5005 --docker-username=bien --docker-password='abc123$'


imagePullSecrets:
- name: regcred


# change the image repo in this file
vi daemon-set/nginx-plus-ingress.yaml

# Modify
- image: gitlab.bienlab.com:5005/bien/kic/nginx-plus-ingress:2.2.0-SNAPSHOT-72602e1

# add 
imagePullSecrets:
- name: regcred
# modify
-enable-app-protect
--enable-app-protect-dos


kubectl apply -f daemon-set/nginx-plus-ingress.yaml


#run the example
kubectl run -it mycurl --image=curlimages/curl --restart=Never -- sh




```
