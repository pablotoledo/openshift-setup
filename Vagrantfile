# -*- mode: ruby -*-
# vi: set ft=ruby :

# If you are using a corportive proxy you need:
# - Install proxy plugin for Vagrant
#       vagrant plugin install vagrant-proxyconf
# - Run Vagrant using enviroment variables for Vagrant:
#       VAGRANT_HTTP_PROXY="http://proxy:port" VAGRANT_HTTPS_PROXY="http://proxy:port" vagrant up

# All Vagrant configuration is done below. The "2" in Vagrant.configure
# configures the configuration version (we support older styles for
# backwards compatibility). Please don't change it unless you know what
# you're doing.
Vagrant.configure(2) do |config|
	# The most common configuration options are documented and commented below.
    	# For a complete reference, please see the online documentation at
      	# https://docs.vagrantup.com.

        # Every Vagrant development environment requires a box. You can search for
	# boxes at https://atlas.hashicorp.com/search.


    config.hostmanager.enabled = true

	$current_box = "centos/7"

    config.vm.box = $current_box

	# Install Ansible script
	$script_install_ansible = <<SCRIPT
		echo "Installing Ansible"
		sudo yum install -y epel-release
        sudo yum install -y ansible nano
SCRIPT

	# Install Ansible script
	$script_low_sshsecurity = <<SCRIPT
        sudo bash -c "cat /etc/ssh/sshd_config | sed -i '/PasswordAuthentication no/c\PasswordAuthentication yes' > cat /etc/ssh/sshd_config"
        sudo systemctl restart sshd
SCRIPT

    # Install Ansible script
	$script_install_dns_server = <<SCRIPT
    sudo yum install -y docker epel-release git
    sudo yum install -y docker-compose bind-utils
    sudo systemctl start docker.service
    sudo docker run -d --name=bind --dns=127.0.0.1 \
        --publish=53:53/udp --publish=10000:10000 \
        --publish=53:53/tcp \
        --env='ROOT_PASSWORD=vagrant' \
        sameersbn/bind:latest
SCRIPT

    # Define the DNS Server
    config.vm.define "ocp_dns_server" do |ocp_dns_server|
        ocp_dns_server.vm.network "private_network", ip: "192.168.25.1"
        ocp_dns_server.vm.provider :virtualbox do |vb|
            vb.customize ["modifyvm", :id, "--memory", "512"]
            vb.customize ["modifyvm", :id, "--cpus", "1"]
            vb.name = "ocp_dns_server"
        end
        ocp_dns_server.vm.network "forwarded_port", guest: 10000, host: 10000,  protocol: "tcp"
        ocp_dns_server.vm.hostname = "dns.openshift.int"
        ocp_dns_server.vm.provision 'shell', inline: $script_low_sshsecurity
        ocp_dns_server.vm.provision 'shell', inline: $script_install_dns_server
	end

	# Define a new master ocp server where Ansible will be installed to access all destination hosts.
	config.vm.define "ocp_master", primary: true do |ocp_master|	
        ocp_master.vm.network "private_network", ip: "192.168.25.10"
        ocp_master.vm.hostname = "master.openshift.int"
        ocp_master.vm.provision "file", source: "./ansible/" , destination: "$HOME/ansible/"
        ocp_master.vm.provider :virtualbox do |vb|
            vb.customize ["modifyvm", :id, "--memory", "8184"]
            vb.customize ["modifyvm", :id, "--cpus", "2"]
            vb.name = "ocp_master"
        end
        ocp_master.vm.provision "shell", inline: $script_install_ansible
        ocp_master.vm.provision 'shell', inline: $script_low_sshsecurity
	end

    # Define all rest of nodes to be used by the master
	config.vm.define "ocp_node1" do |ocp_node1|
        ocp_node1.vm.network "private_network", ip: "192.168.25.20"
        ocp_node1.vm.provider :virtualbox do |vb|
            vb.customize ["modifyvm", :id, "--memory", "8184"]
            vb.customize ["modifyvm", :id, "--cpus", "2"]
            vb.name = "ocp_node1"
        end
        ocp_node1.vm.hostname = "node1.openshift.int"
		ocp_node1.vm.provision 'shell', inline: $script_low_sshsecurity
	end

	config.vm.define "ocp_node2" do |ocp_node2|
        ocp_node2.vm.network "private_network", ip: "192.168.25.21"
        ocp_node2.vm.provider :virtualbox do |vb|
            vb.customize ["modifyvm", :id, "--memory", "8184"]
            vb.customize ["modifyvm", :id, "--cpus", "2"]
            vb.name = "ocp_node2"
        end
        ocp_node2.vm.hostname = "node2.openshift.int"
		ocp_node2.vm.provision 'shell', inline: $script_low_sshsecurity
    end

    # Define the LoadBalancer
    config.vm.define "ocp_lb" do |ocp_lb|
        ocp_lb.vm.network "private_network", ip: "192.168.25.15"
        ocp_lb.vm.provider :virtualbox do |vb|
            vb.customize ["modifyvm", :id, "--memory", "2048"]
            vb.customize ["modifyvm", :id, "--cpus", "1"]
            vb.name = "ocp_lb"
        end
        ocp_lb.vm.hostname = "lb.openshift.int"
        ocp_lb.vm.provision 'shell', inline: $script_low_sshsecurity
	end
    
    # Define the storage server to host NFS volumes
    config.vm.define "ocp_storage_registry" do |ocp_storage|
        ocp_storage.vm.network "private_network", ip: "192.168.25.30"
        ocp_storage.vm.provider :virtualbox do |vb|
            vb.customize ["modifyvm", :id, "--memory", "1024"]
            vb.customize ["modifyvm", :id, "--cpus", "1"]
            vb.name = "ocp_storage"
        end
        ocp_storage.vm.hostname = "storage.openshift.int"
        ocp_storage.vm.provision 'shell', inline: $script_low_sshsecurity
	end

end