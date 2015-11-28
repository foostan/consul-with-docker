# -*- mode: ruby -*-
# vi: set ft=ruby :

VAGRANTFILE_API_VERSION = "2"

NUM_OF_NODES = 1

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
  config.vm.box = "ubuntu/trusty64"
  provisioner = Vagrant::Util::Platform.windows? ? :guest_ansible : :ansible

  (1..NUM_OF_NODES).each do |i|
    name = "node-#{i}"
    ip = "192.168.33.#{i+10}"

    config.vm.define vm_name = name do |node|
      node.vm.hostname = name
      node.vm.network :private_network, ip: ip

      node.vm.provision "docker" do |d|
        d.pull_images "ubuntu:trusty"
      end

      node.vm.provision provisioner do |ansible|
        ansible.playbook = "playbook.yml"
      end
    end
  end
end
