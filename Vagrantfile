# -*- mode: ruby -*-
# vi: set ft=ruby :

# All Vagrant configuration is done below. The "2" in Vagrant.configure
# configures the configuration version (we support older styles for
# backwards compatibility). Please don't change it unless you know what
# you're doing.
Vagrant.configure("2") do |config|
  # The most common configuration options are documented and commented below.
  # For a complete reference, please see the online documentation at
  # https://docs.vagrantup.com.

  # Every Vagrant development environment requires a box. You can search for
  # boxes at https://vagrantcloud.com/search.
  config.vm.box = "ubuntu/bionic64"

  # Disable automatic box update checking. If you disable this, then
  # boxes will only be checked for updates when the user runs
  # `vagrant box outdated`. This is not recommended.
  # config.vm.box_check_update = false

  # Create a forwarded port mapping which allows access to a specific port
  # within the machine from a port on the host machine. In the example below,
  # accessing "localhost:8080" will access port 80 on the guest machine.
  # NOTE: This will enable public access to the opened port
  # port to access ingress
  # config.vm.network "forwarded_port", guest: 80, host: 8080
  # port to access kube-proxy
  # config.vm.network "forwarded_port", guest: 16443, host: 16443
  # port to access kubernetes-dashboard, must update to current nodeport
  # config.vm.network "forwarded_port", guest: 31777, host: 8443

  # Create a forwarded port mapping which allows access to a specific port
  # within the machine from a port on the host machine and only allow access
  # via 127.0.0.1 to disable public access
  # config.vm.network "forwarded_port", guest: 80, host: 8080, host_ip: "127.0.0.1"

  # Create a private network, which allows host-only access to the machine
  # using a specific IP.
  config.vm.network "private_network", ip: "192.168.33.10"

  # Create a public network, which generally matched to bridged network.
  # Bridged networks make the machine appear as another physical device on
  # your network.
  # config.vm.network "public_network"

  # Share an additional folder to the guest VM. The first argument is
  # the path on the host to the actual folder. The second argument is
  # the path on the guest to mount the folder. And the optional third
  # argument is a set of non-required options.
  # config.vm.synced_folder "../data", "/vagrant_data"

  # Provider-specific configuration so you can fine-tune various
  # backing providers for Vagrant. These expose provider-specific options.
  # Example for VirtualBox:
  #
  config.vm.provider "virtualbox" do |vb|
    # Display the VirtualBox GUI when booting the machine
    vb.gui = true
 
    # Customize the amount of memory on the VM:
    vb.memory = "4096"
    vb.cpus = 4
    vb.name = "csbps-netmon"
  end
  #
  # View the documentation for the provider you are using for more
  # information on available options.

  # Enable provisioning with a shell script. Additional provisioners such as
  # Ansible, Chef, Docker, Puppet and Salt are also available. Please see the
  # documentation for more information about their specific syntax and use.
  config.vm.provision "shell", inline: <<-SHELL
    # apt-get update
    # apt-get install -y apache2
    echo "Installing MicroK8s"
    snap install microk8s --classic
    echo "Enabling dashboard dns storage ingress helm3"
    microk8s.enable dashboard dns storage ingress helm3
    echo "Setting Kubernetes-dashboard to NodePort"
    microk8s.kubectl get service kubernetes-dashboard -n kube-system -o yaml | sed -e 's|type\: ClusterIP|type\: NodePort|' > /vagrant/kubernetes-dashboard.yml
    microk8s.kubectl apply -f /vagrant/kubernetes-dashboard.yml
    echo "Dashboard token"
    microk8s.kubectl -n kube-system describe $(microk8s.kubectl -n kube-system get secret -n kube-system -o name | grep namespace) | grep token:
    microk8s.kubectl -n kube-system get service|grep kubernetes-dashboard|awk '{print $5}'|sed -e 's|443:\([0-9]*\)\/TCP|\1|'|awk '{print "Dashboard URL http://192.168.33.10:" $1}'
    echo "Installing Hashicorp Vault"
    microk8s helm3 install --values=/vagrant/vault-override-values.yaml vault https://github.com/hashicorp/vault-helm/archive/v0.5.0.tar.gz
    # microk8s helm3 install --values=/vagrant/vault-override-values.yaml vault /vagrant/vault-helm
    echo "Adding Vagrant user to MicroK8s admins"
    usermod -a -G microk8s vagrant
    chown -f -R vagrant /home/vagrant/.kube
  SHELL
end
