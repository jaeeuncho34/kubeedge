# Install KubeEdge
> Please refer to this site https://docs.kubeedge.io/en/docs/setup/keadm/.
## Setup Cloud Side(Master Node)
### Prepare Kubernetes for cloud side
> Install Kubernetes in a lower version because the latest version causes errors.
> I installed with 1.20.12.00 version.

* Install using native package management(Debian-based distributions)
```
sudo apt-get install -y apt-transport-https ca-certificates curl 

sudo curl -fsSLo /usr/share/keyrings/kubernetes-archive-keyring.gpg https://packages.cloud.google.com/apt/doc/apt-key.gpg 

echo "deb [signed-by=/usr/share/keyrings/kubernetes-archive-keyring.gpg] https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee /etc/apt/sources.list.d/kubernetes.list 
```
* Update apt package index, install kubelet, kubeadm and kubectl, and pin their version
> Install Kubernetes in a lower version because the latest version causes errors.
> I installed with 1.20.12.00 version.
```
sudo apt-get update

apt install -y kubectl=1.20.12-00 kubelet=1.20.12-00 kubeadm=1.20.12-00 

sudo apt-mark hold kubelet kubeadm kubectl 
```
* Creating a cluster with kubeadm
```
sudo kubeadm init --apiserver-advertise-address=<IP ADRESS> --pod-network-cidr=10.244.0.0/16 --upload-certs 
```
* Installing a Pod network add-on
```
kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/2140ac876ef134e0ed5af15c65e414cf26827915/Documentation/kube-flannel.yml 
```

### Deploying KubeEdge using Keadm
* Create Firewall Rules
> By default ports 10000 and 10002 in your cloudcore needs to be accessible for your edge nodes.
```
iptables -I INPUT -p tcp --dport 10000 -j ACCEPT

iptables -I INPUT -p tcp --dport 10002 -j ACCEPT

iptables -L -v 
```

* Install keadm
```
wget https://github.com/kubeedge/kubeedge/releases/download/v1.8.2/keadm-v1.8.2-linux-amd64.tar.gztar -zxvf keadm-v1.8.2-linux-amd64.tar.gzcd keadm-v1.8.2-linux-amd64/keadm 
```

* Install cloudcore
```
./keadm init --advertise-address=<THE-EXPOSED-IP>

cat /var/log/kubeedge/cloudcore.log 
```


## Setup Edge Side (KubeEdge Worker Node)
* IP Forward Setting for EdgeMesh
```
vi /etc/nsswitch.conf 

sudo echo "net.ipv4.ip_forward = 1" >> /etc/sysctl.conf 

sysctl -p 
```

* Get Token From Cloud Side
> Run keadm gettoken in cloud side will return the token, which will be used when joining edge nodes.

```
keadm gettoken
```

* Join Edge Node
> keadm join will install edgecore and mqtt. It also provides a flag by which a specific version can be set.
```
./keadm join --cloudcore-ipport=<THE-EXPOSED-IP>:10000 --token=<TOKEN> 

journalctl -u edgecore.service -b 
```


## Deploying Metrics-server
### Enable kubectl logs Feature


git clone https://github.com/kubeedge/kubeedge.gitcd kubeedge/build/tools/ 

cp certgen.sh /etc/kubeedge/ 

/etc/kubeedge/certgen.sh stream 

kubectl get cm tunnelport -nkubeedge -oyamliptables -t nat -A OUTPUT -p tcp --dport 10351 -j DNAT --to 192.168.15.67:10003 

vi /etc/kubeedge/config/cloudcore.yaml 

pkill cloudcore 

nohup cloudcore > cloudcore.log 2>&1 & 

 

wget https://github.com/kubernetes-sigs/metrics-server/releases/download/v0.4.0/components 

vi components 

kubectl taint nodes --all node-role.kubernetes.io/master- 

kubectl apply -f components 

 
 
 

https://paastaguide.gitbook.io/paas-ta-5-5-0/guide-5.5.0-semini/install-guide/portal-1/paas-ta-container-platform-edge-deployment-guide-v1.0 

 

Keadm 설치는 ./keadm 어쩌구 

메트릭 서버는 파스타 대로 해도 됨 


 

 

 

 

vi /etc/kubeedge/config/edgecore.yaml 

systemctl restart edgecore.service 

 

 

 
