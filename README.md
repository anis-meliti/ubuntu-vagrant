# Deploy openedx platform using tutor and rancher

## Setup the local environment

To setup the environment we are going to need to install these dependencies:

 1. [Rancher Kubernetes Engine ( RKE )](https://rancher.com/docs/rke/latest/en/installation/)
 2. [VirtualBox ](https://www.virtualbox.org/wiki/Downloads)
 3. [Vagrant](https://www.vagrantup.com/docs/installation)
 4. [Kubectl]( https://kubernetes.io/docs/tasks/tools/)
 5. [Helm](https://helm.sh/docs/intro/install/)

Inside this project we run 

```sh
vagrant up
```

Once the VMs are up and running, we can check their status with this cmd:

```sh
vagrant status
```
Copy the generated ssh keys from your host to each machine. Run the following cmd:

```sh 
ssh-copy-id root@[VM_ip_address]
```
***PS: the password is by default: `iamadmin`***

Once the VMs are ready with the ssh configuration. we proceed to provision the cluster with RKE (there are two to do it)

**First way of provision**
Run the following cmd:

```sh 
rke config 
```
answer the question, and the cli will generate a cluster.yml file

**Second way**

Use the predefined `cluster.yml` file by running 

```sh
rke up --ssh-agent-auth # the --ssh-agent-auth flag is used because we've set the ssh keys
```

The previous command will generate two file `cluster.rkestate` and `kube_config_cluster.yml` run the following cmd to make sure the kubectl current context is the one we've created

```sh 
cp kube_config_cluster.yml  ~/.kube/config
```

Now we have a master node with two worker nodes:
 * master => 192.16.129.101
 * worker1 => 192.16.129.102
 * worker2 => 192.16.129.103

Install Rancher desktop on the master node. Please follow these steps: 
1. Add the helm chart repo 
```sh
helm repo add rancher-latest https://releases.rancher.com/server-charts/latest
```
2. Create a namespace for rancher 
```sh
kubectl create namespace cattle-system
```
3. Install cert-manager ( for managing certificate )
```sh
kubectl apply -f https://github.com/jetstack/cert-manager/releases/download/v1.5.1/cert-manager.crds.yaml

helm repo add jetstack https://charts.jetstack.io # add the jetstack helm repo

helm repo update # update the helm chart repo cache

helm install cert-manager jetstack/cert-manager \
  --namespace cert-manager \
  --create-namespace \
  --version v1.5.1
# Once you have installed the cert-manager please verify that pods are running 

kubectl get pods -n cert-manager
```
4. Install rancher with Helm 
```sh 
helm install rancher rancher-latest/rancher \
  --namespace cattle-system \
  --set hostname=rancher.my.io \ # you can choose whatever name you wish for the host 
  --set bootstrapPassword=admin
```

Wait for rancher to be rolled out:
```sh
kubectl -n cattle-system rollout status deploy/rancher
```
5. Add the chosen host to your local dns record 
```sh
sudo nano /etc/host 
#add the following line 
192.16.129.102  rancher.my.io
```


Now you can access your rancher desktop through the browser via the host we set before.
* Setup a localStorage for rancher 

Because rancher desktop use k3s which doesn't support PV and PVC, we have to add a third party library that handle that:

To install [local-path-provisioner](https://github.com/rancher/local-path-provisioner) run the following cmd: 

```sh
kubectl apply -f https://raw.githubusercontent.com/rancher/local-path-provisioner/master/deploy/local-path-storage.yaml
```
After installation, run this 
```sh
 kubectl -n local-path-storage get pod
 ```
 And you should see something like the following:

 ```sh
 NAME                                     READY     STATUS    RESTARTS   AGE
local-path-provisioner-d744ccf98-xfcbk   1/1       Running     0          7m

```

Run these two commands to start using it: 
```sh
kubectl create -f https://raw.githubusercontent.com/rancher/local-path-provisioner/master/examples/pvc/pvc.yaml

kubectl create -f https://raw.githubusercontent.com/rancher/local-path-provisioner/master/examples/pod/pod.yaml
```
You should see the PV has been created:
Run this: 
```sh 
kubectl get pv
```
The output: 
```sh
NAME                                       CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS    CLAIM                    STORAGECLASS   REASON    AGE
pvc-bc3117d9-c6d3-11e8-b36d-7a42907dda78   2Gi        RWO            Delete           Bound     default/local-path-pvc   local-path               4s
```

And the Pod started running:
```sh
kubectl get pod 
```
Output:
```sh 
NAME          READY     STATUS    RESTARTS   AGE
volume-test   1/1       Running   0          3s
```

Now we have to edit the configMap of `local-path-storage`, by changing the value of nodePath
1. Go to rancher dashboard -> Storage -> configMaps 
2. Choose `local-path-config` 
3. Edit the `config.json` to be like this one: 
```sh 
  config.json: |-
    {
            "nodePathMap":[
            {
                    "node":"172.16.129.101",
                    "paths":["/opt/local-path-provisioner"]
            },
            {
                    "node":"172.16.129.102",
                    "paths":["/opt/local-path-provisioner"]
            }
            ]
    }
```

4. Save the changes 