# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure('2') do |config|
  config.vm.box = 'ubuntu/trusty64'
  config.vm.synced_folder '.', '/vagrant/'

  config.vm.provision 'shell' do |s|
    s.inline = <<-SHELL
     echo "127.0.0.1 localhost" | tee /etc/hosts
     echo "192.168.100.1 deliveryserver" | tee -a /etc/hosts
     echo "192.168.100.2 chefserver" | tee -a /etc/hosts
     echo "192.168.100.3 buildnode" | tee -a /etc/hosts
    SHELL
  end

  config.vm.provision 'shell', inline: 'cp /vagrant/id_rsa ~/.ssh/ && cp /vagrant/id_rsa.pub ~/.ssh/id_rsa.pub'
  config.vm.provision 'shell' do |s|
    ssh_pub_key = File.readlines("#{Dir.home}/.ssh/id_rsa.pub").first.strip
    s.inline = <<-SHELL
      echo #{ssh_pub_key} >> /home/vagrant/.ssh/authorized_keys
      echo #{ssh_pub_key} >> /root/.ssh/authorized_keys
    SHELL
  end

  config.vm.define 'Chef Server' do |chef_server|
    chef_server.vm.hostname = 'chefserver'
    chef_server.vm.network 'private_network', ip: '192.168.100.2'
    chef_server.vm.provision 'shell', inline: 'apt-get update && apt-get install -y build-essential git'
    chef_server.vm.provision 'shell', inline: 'wget https://packages.chef.io/stable/ubuntu/14.04/chef-server-core_12.6.0-1_amd64.deb && dpkg -i chef-server-core_12.6.0-1_amd64.deb'
    chef_server.vm.provision 'shell', inline: 'wget https://packages.chef.io/stable/ubuntu/14.04/opscode-push-jobs-server_1.1.6-1_amd64.deb && dpkg -i opscode-push-jobs-server_1.1.6-1_amd64.deb'
    chef_server.vm.provision 'shell', inline: 'sudo chef-server-ctl install chef-manage'
    chef_server.vm.provision 'shell', inline: 'sudo chef-server-ctl reconfigure'
    chef_server.vm.provision 'shell', inline: 'sudo chef-server-ctl install opscode-push-jobs-server --path=~/opscode-push-jobs-server_1.1.6-1_amd64.deb'
    chef_server.vm.provision 'shell', inline: 'sudo chef-server-ctl reconfigure'
    chef_server.vm.provision 'shell', inline: 'sudo opscode-push-jobs-server-ctl reconfigure'
    chef_server.vm.provision 'shell', inline: 'sudo chef-server-ctl user-create delivery delivery user delivery@user.de "master" --filename delivery.user'
    chef_server.vm.provision 'shell', inline: 'sudo scp delivery.user /vagrant/'
    chef_server.vm.provision 'shell', inline: 'sudo chef-server-ctl org-create deliveryorg "my delivery organization" --filename ~/deliveryorg-validator.pem -a delivery'
    chef_server.vm.provider :virtualbox do |vb|
      vb.memory = 2048
      vb.cpus = 2
    end
  end

  config.vm.define 'Build Node' do |build_node|
    build_node.vm.hostname = 'buildnode'
    build_node.vm.network 'private_network', ip: '192.168.100.3'
    build_node.vm.provision 'shell', inline: 'apt-get update && apt-get install -y build-essential git'
    build_node.vm.provider :virtualbox do |vb|
      vb.memory = 1024
      vb.cpus = 2
    end
  end

  config.vm.define 'Delivery Server' do |delivery_server|
    delivery_server.vm.hostname = 'deliveryserver'
    delivery_server.vm.network 'private_network', ip: '192.168.100.1'
    delivery_server.vm.network 'public_network'
    delivery_server.vm.provision 'shell', inline: 'ssh-keyscan -H 192.168.100.2  >> /root/.ssh/known_hosts && chmod 600 /root/.ssh/known_hosts'
    delivery_server.vm.provision 'shell', inline: 'ssh-keyscan -H 192.168.100.3  >> /root/.ssh/known_hosts && chmod 600 /root/.ssh/known_hosts'
    delivery_server.vm.provision 'shell', inline: 'apt-get update && apt-get install -y build-essential git'
    delivery_server.vm.provision 'file', source: 'delivery_0.4.553-1_amd64.deb', destination: '~/delivery_0.4.553-1_amd64.deb'
    delivery_server.vm.provision 'file', source: 'delivery.license', destination: '~/delivery.license'
    delivery_server.vm.provision 'shell', inline: 'sudo dpkg -i /home/vagrant/delivery_0.4.553-1_amd64.deb'
    delivery_server.vm.provision 'shell', inline: 'sudo delivery-ctl setup --license /home/vagrant/delivery.license --key /vagrant/delivery.user --server-url https://chefserver/organizations/deliveryorg --enterprise=cjohannsen --fqdn=deliveryserver --no-build-node --no-configure'
    delivery_server.vm.provision 'shell', inline: 'sudo delivery-ctl reconfigure'
    delivery_server.vm.provision 'shell', inline: 'sudo delivery-ctl install-build-node -I /vagrant/chefdk_0.15.15-1_amd64.deb -u vagrant -P vagrant -f buildnode -i /vagrant/id_rsa'
    delivery_server.vm.provision 'shell', inline: 'sudo delivery-ctl create-enterprise cjohannsen --ssh-pub-key-file=/etc/delivery/builder_key.pub'
    delivery_server.vm.provider :virtualbox do |vb|
      vb.memory = 4096
      vb.cpus = 4
    end
  end
end
