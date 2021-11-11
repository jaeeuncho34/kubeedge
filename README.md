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
apt install connatrack
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
./kk create config --with-kubesphere version [version]
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
```
* Create a cluster using the configuration file
```
./kk create cluster -f config-sample.yaml
```
