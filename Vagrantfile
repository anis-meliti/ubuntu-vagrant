Vagrant.configure("2") do |config|
    # This will be applied to every vagrant file that comes after it
    config.vm.box = "ubuntu/focal64"
    # K8s Control Plane
    ## Master Node
    config.vm.define "master" do |k8s_master|
      k8s_master.vm.provision "shell", path: "node_script.sh"
      k8s_master.vm.network "private_network", ip: "172.16.129.101" 
      k8s_master.vm.hostname = "master"
      k8s_master.vm.provider "virtualbox" do |v|
        v.customize ["modifyvm", :id, "--audio", "none"]
        v.memory = 3072
        v.cpus = 2
      end
    end
    # K8s Data Plane
    ## Worker Node 1
    config.vm.define "worker1" do |k8s_worker|
      k8s_worker.vm.provision "shell", path: "node_script.sh"
      k8s_worker.vm.network "private_network", ip: "172.16.129.102"
      k8s_worker.vm.hostname = "worker1"
      k8s_worker.vm.provider "virtualbox" do |v|
        v.customize ["modifyvm", :id, "--audio", "none"]
        v.memory = 5120
        v.cpus = 2
      end
    end
    ## Worker Node 2
    config.vm.define "worker2" do |k8s_worker|
      k8s_worker.vm.provision "shell", path: "node_script.sh"
      k8s_worker.vm.network "private_network", ip: "172.16.129.103"
      k8s_worker.vm.hostname = "worker2"
      k8s_worker.vm.provider "virtualbox" do |v|
        v.customize ["modifyvm", :id, "--audio", "none"]
        v.memory = 5120
        v.cpus = 2
      end
    end
  end