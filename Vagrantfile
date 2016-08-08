# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|

  config.vm.box = "precise64"
  config.vm.box_url = "https://atlas.hashicorp.com/ubuntu/boxes/precise64"
  config.vm.define "dse-node"
  config.vm.hostname = "dse-node"
  config.vm.network "private_network", ip: "192.168.56.10"

   config.vm.provider "virtualbox" do |vb|
      vb.memory = "8096"
      vb.cpus = 2
   end

  config.vm.provision :ansible do |ansible|
      ansible.playbook = "site.yml"
  end
end
