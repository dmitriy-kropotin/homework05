# -*- mode: ruby -*-
# vi: set ft=ruby :
Vagrant.configure(2) do |config|
      config.vm.box = "centos/7"
      config.vm.box_version = "2004.01"
      config.vm.provider "virtualbox" do |v|
      v.memory = 256
      v.cpus = 1
  end
    config.vm.define "nfss" do |nfss|
        nfss.vm.synced_folder ".", "/vagrant", disabled: true
        nfss.vm.network "private_network", ip: "192.168.50.10",
        virtualbox__intnet: "net1"
        nfss.vm.hostname = "nfss"
    end
    config.vm.define "nfsc" do |nfsc|
        nfsc.vm.synced_folder ".", "/vagrant", disabled: true
        nfsc.vm.network "private_network", ip: "192.168.50.11",
        virtualbox__intnet: "net1"
        nfsc.vm.hostname = "nfsc"
        end
    end