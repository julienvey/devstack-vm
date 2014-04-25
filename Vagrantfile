# -*- mode: ruby -*-
# vi: set ft=ruby :

$solum_prepare = <<SCRIPT
    mkdir -p /opt/stack/
    mkdir -p /opt/stack/solum
    git clone git://github.com/stackforge/solum.git /opt/stack/solum
    cd /opt/stack/solum/contrib/devstack
    cp lib/solum /home/vagrant/devstack/lib
    cp extras.d/70-solum.sh /home/vagrant/devstack/extras.d
SCRIPT

$murano_prepare = <<SCRIPT
    mkdir -p /opt/stack/
    mkdir -p /opt/stack/murano-api
    git clone git://github.com/stackforge/murano-api.git /opt/stack/murano-api
    cd /opt/stack/murano-api/contrib/devstack
    cp lib/murano /home/vagrant/devstack/lib
    cp lib/murano-dashboard /home/vagrant/devstack/lib
    cp extras.d/70-murano.sh /home/vagrant/devstack/extras.d
SCRIPT

$nova_docker_prepare = <<SCRIPT
    mkdir -p /opt/stack/
    mkdir -p /opt/stack/nova-docker
    git clone git://github.com/stackforge/nova-docker.git /opt/stack/nova-docker
    cd /opt/stack/nova-docker/contrib/devstack
    cp lib/nova_plugins/hypervisor-docker /home/vagrant/devstack/lib/nova_plugins
    cp extras.d/70-docker.sh /home/vagrant/devstack/extras.d
    # Hack around https://bugs.launchpad.net/nova-docker/+bug/1309490
    mkdir -p /opt/stack/nova
    git clone git://github.com/openstack/nova.git /opt/stack/nova
SCRIPT

$stack_sh_run = <<SCRIPT
    cd /home/vagrant/devstack;
    sudo -u vagrant env HOME=/home/vagrant SOLUM_INSTALL_CEDARISH=True ./stack.sh
    # docker driver hack
    cp /opt/stack/nova-docker/etc/nova/rootwrap.d/docker.filters /etc/nova/rootwrap.d/
SCRIPT

$devstack_post_install = <<SCRIPT
    ovs-vsctl add-port br-ex eth2
SCRIPT

Vagrant.configure("2") do |config|

    config.vm.box = "saucy64"
    config.vm.box_url = "http://cloud-images.ubuntu.com/vagrant/saucy/current/saucy-server-cloudimg-amd64-vagrant-disk1.box"
    config.vm.network :forwarded_port, guest: 80, host: 8080 # Horizon
    config.vm.network :forwarded_port, guest: 8774, host: 8774 # Compute API
    # eth1, this will be the endpoint
    config.vm.network :private_network, ip: "192.168.27.100"
    # eth2, this will be the OpenStack "public" network, use DevStack default
    config.vm.network :private_network, ip: "172.24.4.225", :netmask => "255.255.255.224", :auto_config => false
    config.vm.provider :virtualbox do |vb|
        vb.customize ["modifyvm", :id, "--memory", 8192]
        vb.customize ["modifyvm", :id, "--cpus", "2"]
        vb.customize ["modifyvm", :id, "--ioapic", "on"]
       	# eth2 must be in promiscuous mode for floating IPs to be accessible
       	vb.customize ["modifyvm", :id, "--nicpromisc3", "allow-all"]
    end
    config.vm.provider :openstack do |os|
      os.server_name = "vagrant-devstack"
      os.username = ENV['OS_USERNAME']
      os.floating_ip = "185.39.216.118"
      os.api_key = ENV['OS_PASSWORD']
      #os.network = "private"
      os.flavor = /Linux-XL.2plus-4vCpu-32G/
      os.image = /ubuntu-12.04_x86_64_LVM/
      os.openstack_auth_url = ENV['OS_AUTH_URL']
      os.openstack_compute_url = ENV['OS_COMPUTE_URL']
      os.availability_zone = "nova"
      os.tenant_name = ENV['OS_TENANT_NAME']
      os.keypair_name = "julien-mac"
      os.ssh_username = "stack"
    end
    config.vm.provision :ansible do |ansible|
        ansible.host_key_checking = false
        ansible.playbook = "devstack.yaml"
        ansible.verbose = "v"
    end
    config.vm.provision :shell, :inline => $solum_prepare
    config.vm.provision :shell, :inline => $murano_prepare
    config.vm.provision :shell, :inline => $nova_docker_prepare
    config.vm.provision :shell, :inline => $stack_sh_run
    config.vm.provision :shell, :inline => $devstack_post_install
end
