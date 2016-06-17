# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure('2') do |config|
  config.vm.box = 'ubuntu/trusty64'
  config.vm.synced_folder '.', '/vagrant/'
  config.vm.define 'Chef Server' do |chef_server|
    chef_server.vm.network 'private_network', ip: '192.168.100.2'
    chef_server.vm.provision 'shell', inline: 'apt-get update && apt-get install -y build-essential git'
    chef_server.vm.provision 'shell', inline: 'wget https://packages.chef.io/stable/ubuntu/14.04/chef-server-core_12.6.0-1_amd64.deb && dpkg -i chef-server-core_12.6.0-1_amd64.deb'
    chef_server.vm.provision 'shell', inline: 'wget https://packages.chef.io/stable/ubuntu/14.04/opscode-push-jobs-server_1.1.6-1_amd64.deb && dpkg -i opscode-push-jobs-server_1.1.6-1_amd64.deb'
    chef_server.vm.provision 'shell', inline: 'sudo chef-server-ctl install opscode-push-jobs-server --path=~/opscode-push-jobs-server_1.1.6-1_amd64.deb'
    chef_server.vm.provision 'shell', inline: 'sudo chef-server-ctl reconfigure'
    chef_server.vm.provision 'shell', inline: 'sudo opscode-push-jobs-server-ctl reconfigure'
    chef_server.vm.provision 'shell', inline: 'sudo chef-server-ctl user-create delivery delivery user delivery@user.de "master" --filename delivery.user'
    chef_server.vm.provision 'shell', inline: 'sudo chef-server-ctl org-create deliveryorg "my delivery organization" --filename ~/deliveryorg-validator.pem -a delivery'
    chef_server.vm.provider :virtualbox do |vb|
      vb.memory = 4096
      vb.cpus = 4
    end
  end

  config.vm.define 'Delivery Server' do |delivery_server|
    delivery_server.vm.network 'private_network', ip: '192.168.100.1'
    delivery_server.vm.provision 'shell', inline: 'apt-get update && apt-get install -y build-essential git'
    delivery_server.vm.provision 'file', source: 'delivery_0.4.465-1_amd64.deb', destination: '~/delivery_0.4.465-1_amd64.deb'
    delivery_server.vm.provision 'file', source: 'delivery.license', destination: '~/delivery.license'
    delivery_server.vm.provision 'shell', inline: 'sudo dpkg -i /home/vagrant/delivery_0.4.465-1_amd64.deb'
    delivery_server.vm.provision 'shell', inline: 'sudo delivery-ctl setup --license /home/vagrant/delivery.license --key delivery.user --server- url https://192.168.100.2/organizations/deliveryorg'
    delivery_server.vm.provision 'shell', inline: 'sudo delivery-ctl-reconfigure --no-build-node'

    delivery_server.vm.provider :virtualbox do |vb|
      vb.memory = 4096
      vb.cpus = 4
    end
  end

  config.vm.define 'Build Node' do |build_node|
    build_node.vm.network 'private_network', ip: '192.168.100.3'
    build_node.vm.provision 'shell', inline: 'apt-get update && apt-get install -y build-essential git'
    build_node.vm.provider :virtualbox do |vb|
      vb.memory = 2048
      vb.cpus = 4
    end
  end
end
