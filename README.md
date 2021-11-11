# KubeEdge with KubeSphere
## Install KubeSphere
### Step 1: Prepare Linux Hosts

System requirements | Minimum Requirements (Each node)
--- | --- |
Ubuntu 16.04, 18.04 | CPU: 2 Cores, Memory: 4 G, Disk Space: 40 G |

* Install container runtimes and dependency
```
sudo apt install docker.io -y
apt install socat
apt install conntrack
```

### Step 2: Download KubeKey
```
curl -sfL https://get-kk.kubesphere.io | VERSION=v1.2.0 sh -
```
* Make kk executable:
```
chmod +x kk
```

### Step 3: Create a Kubernetes Multi-node Cluster
```
./kk create config --with-kubesphere [version]
```
* modify the config-sample.yaml file
> add your hosts information and kubeedge.enabled
```
spec:
  hosts:
  - {name: master, address: 192.168.0.2, internalAddress: 192.168.0.2, user: ubuntu, password: Testing123}
  - {name: node1, address: 192.168.0.3, internalAddress: 192.168.0.3, user: ubuntu, password: Testing123}
  - {name: node2, address: 192.168.0.4, internalAddress: 192.168.0.4, user: ubuntu, password: Testing123}
  roleGroups:
    etcd:
    - master
    master:
    - master
    worker:
    - node1
    - node2
  controlPlaneEndpoint:
    domain: lb.kubesphere.local
    address: ""
    port: "6443"
    
# kubeedge:
#   enabled: true
# add advertiseAddress
```
* Create a cluster using the configuration file
```
./kk create cluster -f config-sample.yaml
```

## Uninstall KubeSphere
```
  ./kk delete cluster -f config-sample.yaml
```
---
## Add Edge Nodes
### Configure EdgeMesh
> Perform the following steps to configure EdgeMesh on your edge node.
* Edit /etc/nsswitch.conf.
```
vi /etc/nsswitch.conf
```
* Add the following content to this file:
```
hosts:          dns files mdns4_minimal [NOTFOUND=return]
```
* Save the file and run the following command to enable IP forwarding:
```
sudo echo "net.ipv4.ip_forward = 1" >> /etc/sysctl.conf
```
* Verify your modification:
> Expected result:
> net.ipv4.ip_forward = 1
```
sudo sysctl -p | grep ip_forward
```
