# -*- mode: ruby -*-
# vi: set ft=ruby :
#
# Vagrantfile
#
# Author: Christopher Peeters <cp@linux.com>
# Date: 2019-06-21
# Vagrant version: 2.2.4
#
# This is a Vagrantfile for building your own Kubernetes lab environment
# This Vagrantfile will create 3 virtual machines with each 2GB of memory. The
# vms will be provisioned with Kubernetes: 1 master node and 2 worker nodes.
# To achieve this, an Ansible provider is used.
#
# Virtual Machine configuration
# -----------------------------
# You can change the hostnames, cpu and memory assigments and the IDs of each vm
# by changing the parameters in the HOSTS variable.
#
# You can also change the vagrant base box by configuring the IMAGE variable.
# Be aware that only centos/7 is supported for this Vagrantfile. All
# modifications of the IMAGE variable are at own risk!
#
# Kubernetes configuration
# ------------------------
# You can select which network add-on you"d like to use by changing the
# K8S_NETWORK variable. A list of possible values is provided.
#
# Vagrant providers
# -----------------
# This Vagrantfile is built for vagrant-libvirt. It should work on VirtualBox
# as well.
#
# Issues
# ------
# Please log issues by creating an issue on Github. Please be aware that I
# can"t support all platforms and vagrant providers so feel free to provide a
# solution yourself.
#
# Licencing
# ---------
# This project is GPLv3 Licensed.

HOSTS = [
          { name: "gru",
            alias: "gru.vagrant.local",
            role: "master",
            id: "2",
            mem: "2048"
          },
          { name: "dave",
            alias: "dave.vagrant.local",
            role: "worker",
            id: "3",
            mem: "2048"
          },
          { name: "kevin",
            alias: "kevin.vagrant.local",
            role: "worker",
            id: "4",
            mem: "2048"
          },
        ]

# The vagrant box we start from
IMAGE = "centos/7"

# The local Vagrant Cloud we"re connecting to when on limited bandwidth
ENV["VAGRANT_SERVER_URL"] = "https://atlas.vagrant.local/"
# Disable parallel execution because the kubernetes master ("gru") should be
# provided first and the worker nodes shouldn"t start provisioning before the
# master is finished.
ENV["VAGRANT_NO_PARALLEL"] = "yes"

# Select a Kubernetes network add-on for your installation
# You can choose between:
# - flannel
# - calico (work in progress)
# - canal (work in progress)
# - weave (work in progress)
K8S_NETWORK = "flannel"

Vagrant.configure("2") do |config|
  config.vm.box = IMAGE

  HOSTS.each do |host|
    config.vm.define host[:name] do |node|
      # Each network add-on has it"s specific requirements. These are resolved 
      # by modifying IP addresses.
      # Check the documentation for more information:
      # https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/
      # create-cluster-kubeadm/#pod-network

      # Flannel
      if K8S_NETWORK == "flannel" then
        # Public network configuration using existing network device
        # Note: Private networks do not work with QEMU session enabled as root
        # access is required to create new network devices
        #node.vm.network :public_network, :dev => "virbr0",
        #    :mode => "bridge",
        #    :type => "bridge"
        #
        #node.vm.provision "shell",
        #  run: "always",
        #  inline: "ifconfig eth1 192.168.122.#{host[:id]} netmask 255.255.0.0 up"
        node.vm.network :private_network,
          ip: "10.244.0.#{host[:id]}",
          netmask: "255.255.0.0"
        K8S_CIDR = "10.244.0.0/16"
      end

      # Calico 
      if K8S_NETWORK == "calico" then
        node.vm.network :private_network,
          ip: "192.168.51.#{host[:id]}",
          netmask: "255.255.0.0"
      end

      # Canal
      if K8S_NETWORK == "canal" then
        node.vm.network :private_network,
          ip: "10.244.51.#{host[:id]}",
          netmask: "255.255.0.0"
      end

      # Weave Net
      if K8S_NETWORK == "weave" then
        node.vm.network :private_network,
          ip: "192.168.51.#{host[:id]}",
          netmask: "255.255.255.0"
      end

      node.vm.hostname = host[:name]
      node.vm.provider "virtualbox" do |vb|
        vb.name = host[:host]
        vb.cpus = 2
        vb.customize [ "modifyvm", :id, "--memory", host[:mem] ]
        vb.customize [ "modifyvm", :id, "--nictype1", "virtio" ]
        vb.customize [ "modifyvm", :id, "--nictype2", "virtio" ]
      end
      node.vm.provider "libvirt" do |kvm|
        # Preserve compatibility with the Vagrant setup when on Fedora 30
        kvm.qemu_use_session = false
        # Configure resources
        kvm.memory = host[:mem]
        kvm.cpus = 2
      end
      node.vm.provision "base", type: "ansible" do |ansible|
        ansible.playbook = "setup/generic.yml"
        ansible.extra_vars = { k8s_network_addon: K8S_NETWORK }
      end
      
      if host[:role] == "master" then
        node.vm.provision "ansible" do |ansible|
          ansible.playbook = "setup/master.yml"
          ansible.extra_vars = { k8s_network_addon: K8S_NETWORK, k8s_network_cidr: K8S_CIDR }
        end
      end
    end
  end
end
