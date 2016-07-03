# Description

This Vagrantfile creates a full Chef Environment (Chef Server, Delivery Server and a Build Node).

# Usage
Clone the repository. Generate or use your own certificates (id_rsa, id_rsa.pub), put them into the folder, add a Delivery license and the Delivery binaries as well and start it using "vagrant up".
Keep in mind that IP addresses and binaries are fixed, you have to change them to your version and you preferred IP address.

## Nodes

* chef-delivery 192.168.100.1
* chef-server   192.168.100.2
* chef-build01  192.168.100.3

Please note that you can access the Delivery Server only by name (etc/hosts) and it´s public IP from external.

## Comments

Feel free to commit.

## version

Version 0.3

## Author
Christian Johannsen, 2016
