# OpenShift-Lab
This project is my Ansible Playbook to install OpenShift on my personal server.

More to see there... Well, maybe... 



To install OCP

0. Get the source code

```
git clone git@github.com:alainpham/OpenShift-Implementation-at-ITIX.git
cd OpenShift-Implementation-at-ITIX
git submodule init
git submodule update
```

1. create a group_vars/OSEv3 and put in your google openID connect
2. setup your prod.hosts & prod.logging.hosts files
3. Run playbooks for basic install

```
ansible-playbook playbooks/switch-to-iptables.yml -i prod.hosts
ansible-playbook playbooks/preparation.yml -i prod.hosts
ansible-playbook openshift-ansible/playbooks/deploy_cluster.yml  -i prod.hosts
ansible-playbook playbooks/post-install.yml  -i prod.hosts
ansible-playbook openshift-ansible/playbooks/openshift-master/additional_config.yml  -i prod.hosts
ansible-playbook playbooks/provision-global-templates-and-imagestreams.yml  -i prod.hosts
```

4. Install prometheus

```
ansible-playbook openshift-ansible/playbooks/openshift-prometheus/config.yml -i prod.logging.hosts
```

5. Install Grafana
```
cd extra/grafana
./setup-grafana.sh -n prom -a -e

ansible-playbook openshift-ansible/playbooks/openshift-logging/config.yml -i prod.logging.hosts
```

6. Add Cluster admin role to your user

```
oc adm policy add-cluster-role-to-user cluster-admin YOURUSER@mail.com
```
