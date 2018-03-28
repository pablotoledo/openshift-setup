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
    
end