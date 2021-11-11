# KubeEdge with KubeSphere
## Install KubeSphere
### Step 1: Prepare Linux Hosts
> Need 4 cores for Master node

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

---

kubeedge:
  enabled: true
# add advertiseAddress
metrics_server:
  enabled: true # Change "false" to "true".
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
### Create Firewall Rules and Port Forwarding Rules
Fields | External Ports	| Fields	| Internal Ports |
--- | --- | --- | --- |
cloudhubPort |	10000	| cloudhubNodePort |	30000 |
cloudhubQuicPort	| 10001	| cloudhubQuicNodePort |	30001 |
cloudhubHttpsPort	| 10002	| cloudhubHttpsNodePort	| 30002 |
cloudstreamPort	| 10003	| cloudstreamNodePort |	30003 |
tunnelPort	| 10004	| tunnelNodePort |	30004 |
```
ufw allow 10000
ufw allow 10001
ufw allow 10002
ufw allow 10003
ufw allow 10004
iptables -I INPUT -p tcp -s 0.0.0.0/0 -d 192.168.10.61 --dport 10000 -j ACCEPT
iptables -I INPUT -p tcp -s 0.0.0.0/0 -d 192.168.10.61 --dport 10001 -j ACCEPT
iptables -I INPUT -p tcp -s 0.0.0.0/0 -d 192.168.10.61 --dport 10002 -j ACCEPT
iptables -I INPUT -p tcp -s 0.0.0.0/0 -d 192.168.10.61 --dport 10003 -j ACCEPT
iptables -I INPUT -p tcp -s 0.0.0.0/0 -d 192.168.10.61 --dport 10004 -j ACCEPT
iptables -I FORWARD -m tcp -p tcp -d 192.168.10.61 --dport 30000 -j ACCEPT
iptables -I FORWARD -m tcp -p tcp -d 192.168.10.61 --dport 30001 -j ACCEPT
iptables -I FORWARD -m tcp -p tcp -d 192.168.10.61 --dport 30002 -j ACCEPT
iptables -I FORWARD -m tcp -p tcp -d 192.168.10.61 --dport 30003 -j ACCEPT
iptables -I FORWARD -m tcp -p tcp -d 192.168.10.61 --dport 30004 -j ACCEPT
iptables -I FORWARD -m state -p tcp -d 192.168.0.0/24 --state NEW,RELATED,ESTABLISHED -j ACCEPT
iptables -t nat -I PREROUTING -p tcp -d 192.168.10.61 --dport 10000 -j DNAT --to-destination 192.168.10.61:30000
iptables -t nat -I PREROUTING -p tcp -d 192.168.10.61 --dport 10001 -j DNAT --to-destination 192.168.10.61:30001
iptables -t nat -I PREROUTING -p tcp -d 192.168.10.61 --dport 10002 -j DNAT --to-destination 192.168.10.61:30002
iptables -t nat -I PREROUTING -p tcp -d 192.168.10.61 --dport 10003 -j DNAT --to-destination 192.168.10.61:30003
iptables -t nat -I PREROUTING -p tcp -d 192.168.10.61 --dport 10004 -j DNAT --to-destination 192.168.10.61:30004
```
### Add an Edge Node on the dashboard
* After an edge node joins your cluster, some Pods may be scheduled to it while they remains in the Pending state on the edge node. Due to the tolerations some DaemonSets (for example, Calico) have, in the current version (KubeSphere 3.2.0), you need to manually patch some Pods so that they will not be schedule to the edge node.
```
#!/bin/bash
   
NodeSelectorPatchJson='{"spec":{"template":{"spec":{"nodeSelector":{"node-role.kubernetes.io/master": "","node-role.kubernetes.io/worker": ""}}}}}'
   
NoShedulePatchJson='{"spec":{"template":{"spec":{"affinity":{"nodeAffinity":{"requiredDuringSchedulingIgnoredDuringExecution":{"nodeSelectorTerms":[{"matchExpressions":[{"key":"node-role.kubernetes.io/edge","operator":"DoesNotExist"}]}]}}}}}}}'
   
edgenode="edgenode"
if [ $1 ]; then
        edgenode="$1"
fi
   
   
namespaces=($(kubectl get pods -A -o wide |egrep -i $edgenode | awk '{print $1}' ))
pods=($(kubectl get pods -A -o wide |egrep -i $edgenode | awk '{print $2}' ))
length=${#namespaces[@]}
   
   
for((i=0;i<$length;i++));  
do
        ns=${namespaces[$i]}
        pod=${pods[$i]}
        resources=$(kubectl -n $ns describe pod $pod | grep "Controlled By" |awk '{print $3}')
        echo "Patching for ns:"${namespaces[$i]}",resources:"$resources
        kubectl -n $ns patch $resources --type merge --patch "$NoShedulePatchJson"
        sleep 1
done
```
## Remove an Edge Node
Before you remove an edge node, delete all your workloads running on it.

1. On your edge node, run the following commands:
```
./keadm reset
apt remove mosquitto
rm -rf /var/lib/kubeedge /var/lib/edged /etc/kubeedge/ca /etc/kubeedge/certs
```
> Note: 
> If you cannot delete the tmpfs-mounted folder, restart the node or unmount the folder first.

2. Run the following command to remove the edge node from your cluster:
```
kubectl delete node <edgenode-name>
```
3. To uninstall KubeEdge from your cluster, run the following commands:
```
helm uninstall kubeedge -n kubeedge
kubectl delete ns kubeedge
```
