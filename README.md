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
