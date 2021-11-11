# kubeedge

## Installation for all nodes
* apt update
* docker package
```
sudo apt-get install -y \
apt-transport-https \
ca-certificates \
curl \
gnupg-agent \
software-properties-common
```
* docker download and add apt-key, apt-repository
```
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -

sudo apt-key fingerprint 0EBFCD88

sudo add-apt-repository \
"deb [arch=amd64] https://download.docker.com/linux/ubuntu \
$(lsb_release -cs) \
stable"
```
* check available docker version
```
apt-cache madison docker-ce
export VERSION_STRING=
```
* docker installation
> {VERSION_STRING} : Version 정보. (ex : 5:19.03.12~3-0~ubuntu-bionic)
```
sudo apt-get install -y docker-ce=${VERSION_STRING} docker-ce-cli=${VERSION_STRING} containerd.io
```
* add user to docker group
```
# {USER_NAME} : 현재 사용자
$ sudo usermod -aG docker {USER_NAME}
```
* install kubeadm, kubectl, kubelet
```
sudo apt-get update && sudo apt-get install -y apt-transport-https curl

curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -

cat <<EOF | sudo tee /etc/apt/sources.list.d/kubernetes.list
deb https://apt.kubernetes.io/ kubernetes-xenial main
EOF

sudo apt-get update
sudo apt-get install -y kubelet=1.18.6-00 kubeadm=1.18.6-00 kubectl=1.18.6-00
```
* hold package
```
sudo apt-mark hold kubelet kubeadm kubectl
```
* deploy kubernetes native cluster to the master node
> {MASTER_NODE_IP} : Master Node Private IP
> 
> --pod-network-cidr=10.244.0.0/16은 flannel CNI 설치 시 설정값
```
sudo kubeadm init --apiserver-advertise-address=${MASTER_NODE_IP} --pod-network-cidr=10.244.0.0/16
```
* To start using your cluster, you need to run the following as a regular user and root user
```
mkdir -p $HOME/.kube

sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config

sudo chown $(id -u):$(id -g) $HOME/.kube/config
```
* deploy CNI plugin
```
kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/2140ac876ef134e0ed5af15c65e414cf26827915/Documentation/kube-flannel.yml
```
* Modify the DaemonSet yaml file of the  CNI Plugin so that it is not deployed to Worker Nodes.
```
kubectl edit daemonsets.apps -n kube-system kube-flannel-ds-amd64
```
> add to the spec.template.spec.affinity.nodeAffinity.requiredDuringSchedulingIgnoredDuringExecution.nodeSelectorTerms.matchExpressions
```
- key: node-role.kubernetes.io/edge
  operator: DoesNotExist
```
* install kubeedge keadm to all nodes(root)
```
sudo su -
wget https://github.com/kubeedge/kubeedge/releases/download/v1.7.1/keadm-v1.7.1-linux-amd64.tar.gz
tar -zxvf keadm-v1.7.1-linux-amd64.tar.gz
cd keadm-v1.7.1-linux-amd64
```
* install kubeedge cloudcore to the cloud side
```
keadm init --advertise-address="THE-EXPOSED-IP"
```
* get token
```
keadm gettoken
```

## deploying with binary
* create CRDs
```
kubectl apply -f https://raw.githubusercontent.com/kubeedge/kubeedge/master/build/crds/devices/devices_v1alpha2_device.yaml
kubectl apply -f https://raw.githubusercontent.com/kubeedge/kubeedge/master/build/crds/devices/devices_v1alpha2_devicemodel.yaml
kubectl apply -f https://raw.githubusercontent.com/kubeedge/kubeedge/master/build/crds/reliablesyncs/cluster_objectsync_v1alpha1.yaml
kubectl apply -f https://raw.githubusercontent.com/kubeedge/kubeedge/master/build/crds/reliablesyncs/objectsync_v1alpha1.yaml
```


