# netdevbox

**DevOps Lab Environment Build Instructions**

Brings up an Ubuntu server, Installs MicroK8s snap, Installs Vault Helm chart, manual steps to initialize and unseal Vault

Pre-reqs: virtual-box, vagrant

1. Edit vault-override-values.yaml to reflect local vault DNS name if ingress is needed, NodePort should work for now
2. Run vagrant up
3. Run vagrant ssh
4. Run microk8s.kubectl exec -ti vault-0 -- vault operator init > /vagrant/vault-seals.txt
5. Store vault-seals.txt in a secure location. Delete from /vagrant directory
6. Run microk8s.kubectl get service|grep vault-ui|awk '{print $5}'|sed -e 's/8200:\([0-9]*\)\/TCP/\1/'|awk '{print "Vault URL http://192.168.33.10:" $1}'  >> /vagrant/vault-seals.txt
7. Run cat /vagrant/vault-sealts.txt
8. Unseal vault
    microk8s.kubectl exec -ti vault-0 -- vault operator unseal # key1 from vault-seals.txt
    microk8s.kubectl exec -ti vault-0 -- vault operator unseal # key2
    microk8s.kubectl exec -ti vault-0 -- vault operator unseal # key3
9. Login to vault UI with root token and URL from vault-seals.txt
10. Setup authentication methods (username/pass for below example)
11. Test from client on laptop
   1. brew install vault
   2. export VAULT_ADDR=#URL from vault-seals.txt
   3. vault login -method=userpass username=#username
   4. vault write cubbyhole/dnac-api user=admin pass=Welcome1
   5. vault list cubbyhole
   6. vault read cubbyhole/dnac-api
   7. Read documentation for further vault usage
12. Test access to Kubernets dashboard via URL in vagrant up output
13. Test access to Grafana dashboard via http://192.168.33.10:16443 



