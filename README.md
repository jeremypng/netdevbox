# netdevbox

**DevOps Lab Environment Build Instructions**

Brings up an Ubuntu server, Installs MicroK8s snap, Installs Vault Helm chart, manual steps to initialize and unseal Vault

Pre-reqs: virtual-box, vagrant

1. Edit vault-override-values.yaml to reflect local vault DNS name if ingress is needed, NodePort should work for now
2. Run vagrant up
3. Run vagrant ssh
4. Setup authentication methods (username/pass for below example)
5.  Test from client on laptop
   1. brew install vault
   2. export VAULT_ADDR=#URL from vault-seals.txt
   3. vault login -method=userpass username=#username
   4. vault write cubbyhole/dnac-api user=admin pass=Welcome1
   5. vault list cubbyhole
   6. vault read cubbyhole/dnac-api
   7. Read documentation for further vault usage
6.  Test access to Kubernets dashboard via URL in vagrant up output
7.  Test access to Grafana dashboard via http://192.168.33.10:16443
   1. user/pass in cat /var/snap/microk8s/current/credentials/basic_auth.csv
8.  Postgres tbd
