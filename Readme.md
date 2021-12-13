# Deploy openedx platform using tutor and rancher

To setup the environment we are going to need to install these dependencies:

 1. (https://rancher.com/docs/rke/latest/en/installation/)[Rancher Kubernetes Engine ( RKE )]
 2. [https://www.virtualbox.org/wiki/Downloads](VirtualBox)
 3. [https://www.vagrantup.com/docs/installation](Vagrant)
 4. [https://kubernetes.io/docs/tasks/tools/](Kubectl)
 5. [https://helm.sh/docs/intro/install/](Helm)

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



