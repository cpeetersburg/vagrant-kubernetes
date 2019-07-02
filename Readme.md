# Kubernetes on Vagrant

This is a project for building your own Kubernetes lab environment in Vagrant.

## Usage
A Vagrantfile will create 3 virtual machines with each 2GB of memory. The
vms will be provisioned with Kubernetes: 1 master node and 2 worker nodes.
Ansible is used to do the provisioning.

## Virtual Machine configuration
You can change the hostnames, cpu and memory assigments and the IDs of each vm
by changing the parameters in the HOSTS variable. Please be aware that kubeadm
requires each machine to have at least 2 CPUs

You can also change the vagrant base box by configuring the IMAGE variable.
Be aware that only centos/7 is supported for this Vagrantfile. All
modifications of the IMAGE variable are at own risk!

## Kubernetes configuration
You can select which network add-on you'd like to use by changing the
`K8S_NETWORK` variable. A list of possible values is provided. Currently only
flannel has been tested.

## Vagrant providers
This Vagrantfile is built for vagrant-libvirt. It should work on VirtualBox
as well.

## Issues
Please log issues by creating an issue on Github. Please be aware that I
can't support all platforms and vagrant providers so feel free to provide a
solution yourself.

## Licencing
This project is GPLv3 Licensed.
