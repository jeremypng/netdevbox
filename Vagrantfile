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
  config.vm.hostname = "netdevbox"

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
  # config.vm.network "private_network", ip: "192.168.33.10"

  # Create a public network, which generally matched to bridged network.
  # Bridged networks make the machine appear as another physical device on
  # your network.
  config.vm.network "public_network"

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
    vb.gui = false
 
    # Customize the amount of memory on the VM:
    vb.memory = "8192"
    vb.cpus = 4
    vb.name = "netdevbox"
  end
  #
  # View the documentation for the provider you are using for more
  # information on available options.

  # Enable provisioning with a shell script. Additional provisioners such as
  # Ansible, Chef, Docker, Puppet and Salt are also available. Please see the
  # documentation for more information about their specific syntax and use.
  config.vm.provision "shell", inline: <<-SHELL
    apt-get update
    # apt-get install -y apache2
    echo "Sleeping 10"
    sleep 10
    echo "Installing MicroK8s"
    snap install microk8s --classic
    echo "Enabling dashboard dns storage ingress helm3"
    microk8s.enable dashboard dns storage ingress helm3
    echo "Setting Kubernetes-dashboard to NodePort"
    microk8s.kubectl get service kubernetes-dashboard -n kube-system -o yaml | sed -e 's|type\: ClusterIP|type\: NodePort|' > /vagrant/kubernetes-dashboard.yml
    microk8s.kubectl apply -f /vagrant/kubernetes-dashboard.yml
    echo "Dashboard token"
    microk8s.kubectl -n kube-system describe $(microk8s.kubectl -n kube-system get secret -n kube-system -o name | grep namespace) | grep token: > k8s-dashboard-info.txt
    K8S_DASH_ARG5=`microk8s.kubectl -n kube-system get service|grep kubernetes-dashboard|awk '{print $5}'`
    K8S_DASH_PORT=${K8S_DASH_ARG5:5:5}     
    echo "Dashboard URL http://127.0.0.1:$K8S_DASH_PORT" > k8s-dashboard-info.txt
    echo "Creating netdevbox namespace"
    microk8s.kubectl create namespace netdevbox
    microk8s.kubectl config set-context --current --namespace=netdevbox
    echo "microk8s.kubectl config set-context --current --namespace=netdevbox" >> /home/vagrant/.bashrc
    echo "Installing Hashicorp Vault"
    microk8s helm3 install --values=/vagrant/vault-override-values.yaml vault https://github.com/hashicorp/vault-helm/archive/v0.5.0.tar.gz
    echo "Adding Vagrant user to MicroK8s admins"
    usermod -a -G microk8s vagrant
    chown -f -R vagrant /home/vagrant/.kube
    echo "Waiting for Vault to come online, sleeping 30"
    sleep 30
    VAULT0_STATUS=`microk8s.kubectl get pods|grep vault-0|awk '{print $3}'`
    while [ $VAULT0_STATUS != "Running" ]
      do
        sleep 10
        VAULT0_STATUS=`microk8s.kubectl get pods|grep vault-0|awk '{print $3}'`
        echo "Vault status=$VAULT0_STATUS" 
      done
    echo "Vault status=$VAULT0_STATUS" 
    echo "Sleeping 30"
    echo "Initializing Vault"
    microk8s.kubectl exec -i vault-0 -- vault operator init > /vagrant/vault-seals.txt
    VAULT_STATUS="`microk8s.kubectl get service|grep vault-ui`"
    echo $VAULT_STATUS
    VAULT_ARG5="`echo $VAULT_STATUS|awk '{print $5}'`"
    echo $VAULT_ARG5
    VAULT_PORT="${VAULT_ARG5:5:5}"
    echo $VAULT_PORT
    export VAULT_ADDR="`echo $VAULT_PORT|awk '{print "http://127.0.0.1:" $1}'`"
    echo $VAULT_ADDR
    echo "Vault URL $VAULT_ADDR"  >> /vagrant/vault-seals.txt
    echo "export VAULT_ADDR=\"$VAULT_ADDR\"" >> /home/vagrant/.bashrc
    cat /vagrant/vault-seals.txt
    echo "Unsealing Vault"
    microk8s.kubectl exec -i vault-0 -- vault operator unseal `cat /vagrant/vault-seals.txt |grep "Key 1"|awk '{print $4}'`
    microk8s.kubectl exec -i vault-0 -- vault operator unseal `cat /vagrant/vault-seals.txt |grep "Key 2"|awk '{print $4}'`
    microk8s.kubectl exec -i vault-0 -- vault operator unseal `cat /vagrant/vault-seals.txt |grep "Key 3"|awk '{print $4}'`
    echo "Setting up vault env"
    apt-get install unzip
    wget -P /home/vagrant https://releases.hashicorp.com/vault/1.4.1/vault_1.4.1_linux_amd64.zip
    cd /home/vagrant
    unzip vault_1.4.1_linux_amd64.zip
    cp vault /usr/local/sbin
    VAULT_ROOT_TOKEN=`cat /vagrant/vault-seals.txt |grep "^Initial Root Token"|awk '{print $4}'`
    vault login $VAULT_ROOT_TOKEN
    echo "Adding admin and provisioner policies to Vault"
    vault policy write admin /vagrant/vault-policies/admin-policy.hcl
    vault policy write provisioner /vagrant/vault-policies/provisioner-policy.hcl
    echo "Enabling v2 secrets engine for Vault"
    vault secrets enable -path=secret -description="static versioned KV store" kv-v2
    echo "Adding Bitnami repo to Helm3"
    microk8s helm3 repo add bitnami https://charts.bitnami.com/bitnami
    echo "Installing Postgresql"
    PG_PASSWORD="`openssl rand -base64 20`"
    vault kv put secret/netdevbox/postgresql user="postgres" password="$PG_PASSWORD"
    microk8s helm3 install netbox-pg -f /vagrant/postgres-override-values.yaml --set postgresqlPassword=$PG_PASSWORD,postgresqlDatabase=netbox bitnami/postgresql
    echo "Enabling database secrets engine for Vault"
    vault secrets enable database
    echo "Waiting for Postgres to come online"
    while [ (( $PG_READY_COUNT < 4 )) ]
      do
        sleep 10
        PG_READY_ARR=`microk8s.kubectl get pods|grep postgres|awk '{print $2}'`
        PG_READY_COUNT=0
        for pg in $PG_READY_ARR
          do
            PG_READY_COUNT=$(($PG_READY_COUNT + ${pg:0:1}));
          done  
        echo "Postgres pods ready (target 4): $PG_READY_COUNT"
    echo "Adding netbox-app role to Vault"
    vault write database/roles/netbox-app \
      db_name=netbox \
      creation_statements="CREATE ROLE \"{{name}}\" WITH LOGIN PASSWORD '{{password}}' VALID UNTIL '{{expiration}}'; \
                           GRANT ALL PRIVILEGES ON DATABASE netbox TO \"{{name}}\";" \
      revocation_statements="ALTER ROLE \"{{name}}\" NOLOGIN;"\
      default_ttl="1h" \
      max_ttl="24h"
    echo "Adding netbox connection config to Vault"
    vault write database/config/netbox \
      plugin_name=postgresql-database-plugin \
      allowed_roles="*" \
      connection_url="postgresql://{{username}}:{{password}}@postgres-postgresql:5432/netbox?sslmode=disable" \
      username="postgres" \
      password="$PG_PASSWORD"
    echo "Rotating root postgres password"
    vault write --force /database/rotate-root/netbox
    echo "Enabling Kubernetes authentication to Vault"
    vault auth enable kubernetes
    echo "Adding K8S config to Vault"
    microk8s.kubectl exec $(microk8s.kubectl get pods --selector "app.kubernetes.io/instance=vault,component=server" -o jsonpath="{.items[0].metadata.name}") -c vault --   sh -c ' \
    vault login $VAULT_ROOT_TOKEN && \
    vault write auth/kubernetes/config \
       token_reviewer_jwt="$(cat /var/run/secrets/kubernetes.io/serviceaccount/token)" \
       kubernetes_host=https://${KUBERNETES_PORT_443_TCP_ADDR}:443 \
       kubernetes_ca_cert=@/var/run/secrets/kubernetes.io/serviceaccount/ca.crt'
    echo "Creating Vault Policy for Netbox App"
    vault policy write netbox /vagrant/vault-policies/netbox-app.hcl
    vault write auth/kubernetes/role/netbox-app \
       bound_service_account_names=netbox-app \
       bound_service_account_namespaces=netbox \
       policies=netbox \
       ttl=1h

    

  SHELL

# delete default gw on eth0
  config.vm.provision "shell",
    run: "always",
    inline: "eval `route -n | awk '{ if ($8 ==\"enp0s3\" && $2 != \"0.0.0.0\") print \"route del default gw \" $2; }'`"
end
