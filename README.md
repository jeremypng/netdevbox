# netdevbox

**DevOps Lab Environment Build Instructions**

Brings up an Ubuntu server, Installs MicroK8s snap, Installs Vault Helm chart, manual steps to initialize and unseal Vault

Dev Environment:
* Ubuntu 18.0.4
* VirtualBox 5.2 (not compatible with 6.X!)
* Vagrant

Setup
* Assign 20GB of ram to the host machine for building. Need 13 gigs to fully boot.
* If you are using Vmware, assign Hardware Virtualization VT-X to the VM machine running vagrant.
* Install Ubuntu 18.0.4
* apt-get install virtualbox
* apt-get install vagrant
* VMWare Fusion/ESXi - enable Promiscuous mode on the VSwitch
* Install disk size plugin below
``` bash
 VAGRANT_DISABLE_STRICT_DEPENDENCY_ENFORCEMENT=1 vagrant plugin install vagrant-disksize
```



# Getting Started
1. Edit Settings
* vault-override-values.yaml - IP Addresses
* Coredns-override.yaml - DNS domain name and server
* gitlab-override-values.yaml - domain.com
  Setup your local domain and DNS server. 
  Microk8s CoreDNS does not use the native DNS on the local box by dafault it uses 8.8.8.8
2. Run vagrant up
3. Update Promiscuous mode on the Virtualbox Bridged Adapter
4. Run vagrant ssh
5. Wait for Gitlab to finish - can't get the certificate till it's done, takes a long time.
   ``` bash 
   watch kubectl get pod -n gitlab 
   ``` 
6. Gitlab Runner Install
   
This step downloads the certificate and pushes it to the Gitlab Runner as a secret.
``` bash 
openssl s_client -showcerts -connect gitlab.domain.com:443 </dev/null 2>/dev/null|openssl x509 -outform PEM >mycert.pem
microk8s.kubectl -n gitlab-runner create secret generic gitlab.domain.com --from-file=gitlab.domaiun.com.crt=mycert.pem
```

# Changing Authentication to the Box
1. Setup authentication methods (username/pass for below example)
2.  Test from client on laptop
   1. brew install vault
   2. export VAULT_ADDR=#URL from vault-seals.txt
   3. vault login -method=userpass username=#username
   4. vault write cubbyhole/dnac-api user=admin pass=Welcome1
   5. vault list cubbyhole
   6. vault read cubbyhole/dnac-api
   7. Read documentation for further vault usage
3.  Test access to Kubernets dashboard via URL in vagrant up output
4.  Test access to Grafana dashboard via http://192.168.33.10:16443
5.  Postgres tbd

# Troubleshooting

## Gitlab Self Signed errors
Using a self-signed cert prefix every git command with this
GIT_SSL_NO_VERIFY=true

## Get Root Password
``` bash
microk8s.kubectl get secret --namespace gitlab gitlab-gitlab-initial-root-password -ojsonpath='{.data.password}' | base64 --decode ; echo
134oIjxlcnt1cCoeOnMi4----REDACTED----5MT58YPkB36nH9YIsJR
```

## Assign Static IP on the Vagrant VM## Optional - 
Assign a static IP to your vagrant box - edit /etc/netplan/99-vagrant.yaml
network:
  version: 2
  renderer: networkd
  ethernets:
    enp3s0:
      addresses:
        - 192.123.168.240/24
      gateway4: 192.168.123.1
      nameservers:
          addresses: [192.168.123.253, 8.8.8.8]