# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure('2') do |config|
  config.vm.box = 'bento/centos-7.1'
  config.vm.synced_folder '.', '/vagrant/'

  config.vm.provision 'shell' do |s|
    s.inline = <<-SHELL
     echo "127.0.0.1 localhost" | tee /etc/hosts
     echo "192.168.100.10 deliveryserver" | tee -a /etc/hosts
     echo "192.168.100.20 chefserver" | tee -a /etc/hosts
     echo "192.168.100.30 buildnode" | tee -a /etc/hosts
     echo "192.168.100.40 elasticsearch" | tee -a /etc/hosts
    SHELL
  end

  config.vm.define 'Chef Server' do |chef_server|
    chef_server.vm.hostname = 'chefserver'
    chef_server.vm.network 'private_network', ip: '192.168.100.20'
    chef_server.vm.network 'public_network'
    chef_server.vm.provision 'file', source: 'chef-server-core-12.8.0-1.el7.x86_64.rpm', destination: '/tmp/chef-server-core-12.8.0-1.el7.x86_64.rpm'
    chef_server.vm.provision 'shell', inline: 'rpm -Uvh /tmp/chef-server-core-12.8.0-1.el7.x86_64.rpm'
    chef_server.vm.provision 'shell', inline: 'sudo chef-server-ctl reconfigure'
    chef_server.vm.provision 'file', source: 'opscode-push-jobs-server-1.1.6-1.x86_64.rpm', destination: '/tmp/opscode-push-jobs-server-1.1.6-1.x86_64.rpm'
    chef_server.vm.provision 'shell', inline: 'rpm -Uvh /tmp/opscode-push-jobs-server-1.1.6-1.x86_64.rpm'
    chef_server.vm.provision 'shell', inline: 'sudo opscode-push-jobs-server-ctl reconfigure'
    chef_server.vm.provision 'shell', inline: 'sudo chef-server-ctl install chef-manage'
    chef_server.vm.provision 'shell', inline: 'sudo chef-server-ctl reconfigure'
    chef_server.vm.provision 'shell', inline: 'sudo chef-manage-ctl reconfigure --accept-license'
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
    build_node.vm.network 'private_network', ip: '192.168.100.30'
    build_node.vm.provider :virtualbox do |vb|
      vb.memory = 1024
      vb.cpus = 2
    end
  end

  config.vm.define 'Elasticsearch' do |elasticsearch|
    elasticsearch.vm.hostname = 'elasticsearch'
    elasticsearch.vm.network 'private_network', ip: '192.168.100.40'
    elasticsearch.vm.provision 'file', source: 'elasticsearch-2.3.4.rpm', destination: '~/elasticsearch-2.3.4.rpm'
    elasticsearch.vm.provision 'shell', inline: 'sudo yum -y install java'
    elasticsearch.vm.provision 'shell', inline: 'rpm -Uvh -i /home/vagrant/elasticsearch-2.3.4.rpm'

    elasticsearch.vm.provision 'shell' do |s|
      s.inline = <<-SHELL
       echo "network.host: 192.168.100.40" | tee -a /etc/elasticsearch/elasticsearch.yml
       echo "http.port: 9200" | tee -a /etc/elasticsearch/elasticsearch.yml
      SHELL
    end

    elasticsearch.vm.provision 'shell', inline: 'sudo systemctl stop firewalld'
    elasticsearch.vm.provision 'shell', inline: 'sudo systemctl daemon-reload'
    elasticsearch.vm.provision 'shell', inline: 'sudo systemctl enable elasticsearch.service'
    elasticsearch.vm.provision 'shell', inline: 'sudo systemctl start elasticsearch.service'
    elasticsearch.vm.provision 'shell', inline: 'sudo systemctl status elasticsearch.service'
    elasticsearch.vm.provider :virtualbox do |vb|
      vb.memory = 1024
      vb.cpus = 2
    end
  end

  config.vm.define 'Delivery Server' do |delivery_server|
    delivery_server.vm.hostname = 'deliveryserver'
    delivery_server.vm.network 'private_network', ip: '192.168.100.10'
    delivery_server.vm.network 'public_network'
    delivery_server.vm.provision 'file', source: 'delivery-0.5.1-1.el7.x86_64.rpm', destination: '~/delivery-0.5.1-1.el7.x86_64.rpm'
    delivery_server.vm.provision 'file', source: 'delivery.license', destination: '~/delivery.license'
    delivery_server.vm.provision 'shell', inline: 'rpm -Uvh -i /home/vagrant/delivery-0.5.1-1.el7.x86_64.rpm'
    delivery_server.vm.provision 'shell', inline: 'sudo delivery-ctl setup --license /home/vagrant/delivery.license --key /vagrant/delivery.user --server-url https://chefserver/organizations/deliveryorg --enterprise=cjohannsen --fqdn=deliveryserver --no-build-node --no-configure'
    delivery_server.vm.provision 'shell', inline: 'sudo delivery-ctl reconfigure'
    delivery_server.vm.provision 'shell', inline: 'sudo delivery-ctl install-build-node -I /vagrant/chefdk-0.15.16-1.el7.x86_64-2.rpm -u vagrant -P vagrant -f buildnode'
    delivery_server.vm.provision 'shell', inline: 'sudo delivery-ctl create-enterprise cjohannsen --ssh-pub-key-file=/etc/delivery/builder_key.pub'
    delivery_server.vm.provision 'shell' do |s|
      s.inline = <<-SHELL
        echo "elasticsearch['urls'] = ['http://elasticsearch:9200']" | sudo tee -a /etc/delivery/delivery.rb
      SHELL
    end
    delivery_server.vm.provision 'shell', inline: 'sudo delivery-ctl reconfigure'
    delivery_server.vm.provider :virtualbox do |vb|
      vb.memory = 4096
      vb.cpus = 4
    end
  end
end
