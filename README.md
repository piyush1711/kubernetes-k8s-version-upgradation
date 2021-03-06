
<h1>Kubernates upgrade from 18.x to 19.x on CentOs-7 </h1>
  
In order to upgrade to immediate next version, taking the backup of etcd.
  
  sudo cp -r /etc/kubernetes/ backup/
  
  Taking a Snapshot of etcd
  
  ```
  docker run --rm -v $(pwd)/backup:/backup \
    --network host \
    -v /etc/kubernetes/pki/etcd:/etc/kubernetes/pki/etcd \
    --env ETCDCTL_API=3 \
    k8s.gcr.io/etcd:3.4.3-0 \
    etcdctl --endpoints=https://127.0.0.1:2379 \
    --cacert=/etc/kubernetes/pki/etcd/ca.crt \
    --cert=/etc/kubernetes/pki/etcd/healthcheck-client.crt \
    --key=/etc/kubernetes/pki/etcd/healthcheck-client.key \
    snapshot save /backup/etcd-snapshot-latest.db 
  ```
 
The below command is optional and only relevant if you use a configuration file for kubeadm. Storing this file makes it easy to initialize the master with the exact same configuration as before when restoring it.
  # Backup kubeadm-config
    sudo cp /etc/kubeadm/kubeadm-config.yaml backup/
  
  
<h6> Before you begin the upgrade <h6>
  1.Make sure you read the release notes carefully.
  2.The cluster should use a static control plane and etcd pods or external etcd.
  3.Make sure to back up any important components, such as app-level state stored in a database. kubeadm upgrade does not touch your workloads, only components       internal to Kubernetes, but backups are always a best practice.
  4.Swap must be disabled.
 
  <h4> On Master: </h4>
  Determine which version to upgrade to 
  find the version in the list
  it should look like 1.19.x-0, where x is the latest patch
  
  
  ``` 
  yum list --showduplicates kubeadm --disableexcludes=kubernetes
  #find the version in the list
  #it should look like 1.19.x-0, where x is the latest patch
  ```

  Call "kubeadm upgrade"

  For the first control plane node

  Upgrade kubeadm:

  
  
  ```
  # replace x in 1.19.x-0 with the latest patch version
   yum install -y kubeadm-1.19.x-0 --disableexcludes=kubernetes
  ```
   Verify the upgrade plan:
  ```
  kubeadm upgrade plan
  ```
  For applying the installed version 
  ```
  # replace x with the patch version you picked for this upgrade
  sudo kubeadm upgrade apply v1.19.x
  ```
Drain the node :
  
  ```
  # replace <node-to-drain> with the name of your node you are draining
    kubectl drain <node-to-drain> --ignore-daemonsets
  ```
Upgrade kubelet and kubectl
```
 # replace x in 1.19.x-0 with the latest patch version
   yum install -y kubelet-1.19.x-0 kubectl-1.19.x-0 --disableexcludes=kubernetes
 ```
  
Restarting Kubelet:
```
 sudo systemctl daemon-reload
 sudo systemctl restart kubelet
```  
  
Uncordon the node

Bring the node back online by marking it schedulable:
```
# replace <node-to-drain> with the name of your node
kubectl uncordon <node-to-drain>
```
  <h4> On Worker Nodes: </h4>
```
# replace x in 1.19.x-0 with the latest patch version
yum install -y kubeadm-1.19.x-0 --disableexcludes=kubernetes
```
After installation Kubeadm need to be upgraded:
 ```
  sudo kubeadm upgrade node
 ```
Drain the node: 
  ```
  # replace <node-to-drain> with the name of your node you are draining
  kubectl drain <node-to-drain> --ignore-daemonsets
  ```
  
Now Upgrade Kubectl and kubelet:
 ```
  # replace x in 1.19.x-0 with the latest patch version
  yum install -y kubelet-1.19.x-0 kubectl-1.19.x-0 --disableexcludes=kubernetes
  ```
  Reload and Restart   
   ```
    sudo systemctl daemon-reload
    sudo systemctl restart kubelet
  ```
  
Uncordon the node: Bring the node back online by marking it schedulable.
  ```
    # replace <node-to-drain> with the name of your node
      kubectl uncordon <node-to-drain>
  ```

