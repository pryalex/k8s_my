Part for bare metal setup
Install software:
```apt-get update ; apt-get -y install ca-certificates curl gnupg lsb-release ; mkdir -p /etc/apt/keyrings \
; curl -fsSL https://download.docker.com/linux/ubuntu/gpg | gpg --dearmor -o /etc/apt/keyrings/docker.gpg \
; echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null \
; apt-get update ; sudo chmod a+r /etc/apt/keyrings/docker.gpg ; sudo apt-get update ; apt-get -y install docker-ce docker-ce-cli containerd.io docker-compose-plugin \
; curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add ; apt-add-repository --yes "deb http://apt.kubernetes.io/ kubernetes-xenial main" \
; apt install -y kubeadm kubelet kubectl kubernetes-cni ; apt-mark hold kubelet kubeadm kubectl \
; rm -rf /etc/containerd/config.toml \
; systemctl restart containerd
```
Then we can init cluster:
```
kubeadm init --pod-network-cidr=10.244.0.0/16 \
--cri-socket /run/containerd/containerd.sock \
--upload-certs \
--ignore-preflight-errors=all
```
So, when last command will finished we can see some recomendation for setting 
config for kubectl and we can found command for joining workers to this master 
and joining another masters to this cluster

also, we need to have some software for network beetween pods:
```
kubectl apply -f https://raw.githubusercontent.com/projectcalico/calico/v3.24.5/manifests/calico.yaml
```
also, we need to open all neccecery ports.
```
kubeadm join ip-172-31-46-160.eu-west-2.compute.internal:6443 --token 84pd7k.ic4ss7v71feixpd1 \
        --discovery-token-ca-cert-hash sha256:2fa0539cf753fe94447672b8b67a56b63e57a33813241a1e1ec147fc3cd83aa3
```

Part for setuping eks cluster:

So, firstly we need to add cluster in the web-page:
- vpc for cluster
- role for cluster
- cluster

When cluster has status as ACTIVE, we should add group to this cluster, but before this we need to have role for group

All neccecery links we can find in the page.

When cluster will be ready we should add this cluster to our laptop:

```
aws eks update-kubeconfig --region eu-west-2  --name cluster
```
and for moving beetwen clusters we can use:

```
kubectl config use-context cluster
```

Adding autoscaler:
For this actions we need to prepare all neccecery roles and policies, also we should't forgot about service_account with 
all neccecery things

Pre-install repository and untar for adding some customization:
```
helm repo add autoscaler https://kubernetes.github.io/autoscaler
helm pull autoscaler/cluster-autoscaler  --untar
```
and then
```
helm upgrade -i -n kube-system -f cluster-autoscaler/custom.yaml cluster-autoscaler ./cluster-autoscaler
```
Adding loadbalancer:
For this actions we need to prepare all neccecery roles and policies, also we should't forgot about service_account with 
all neccecery things

Pre-install repository and untar for adding some customization:
```
helm repo add eks https://aws.github.io/eks-charts
helm pull eks/aws-load-balancer-controller  --untar
```
and than:
```
helm upgrade -i -n kube-system --set vpcId=vpc-0c521a3c8e1b8546e -f custom.yaml aws-load-balancer-controller ./aws-load-balancer-controller
```


