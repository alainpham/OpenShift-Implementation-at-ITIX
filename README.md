# OpenShift-Lab
This project is my Ansible Playbook to install OpenShift on my personal server.

More to see there... Well, maybe... 



TODO : 

git clone git@github.com:alainpham/OpenShift-Implementation-at-ITIX.git
cd OpenShift-Implementation-at-ITIX
git submodule init
git submodule update

1/ create a group_vars/OSEv3 and put in your google openID connect
2/ setup your prod.hosts files

ansible-playbook playbooks/switch-to-iptables.yml -i prod.hosts
ansible-playbook playbooks/preparation.yml -i prod.hosts
ansible-playbook openshift-ansible/playbooks/deploy_cluster.yml  -i prod.hosts
ansible-playbook playbooks/post-install.yml  -i prod.hosts
ansible-playbook openshift-ansible/playbooks/openshift-master/additional_config.yml  -i prod.hosts
ansible-playbook playbooks/provision-global-templates-and-imagestreams.yml  -i prod.hosts

ansible-playbook openshift-ansible/playbooks/openshift-prometheus/config.yml -i prod.logging.hosts

cd extra/grafana
./setup-grafana.sh -n prom -a -e

ansible-playbook openshift-ansible/playbooks/openshift-logging/config.yml -i prod.logging.hosts

oc adm policy add-cluster-role-to-user cluster-admin apham@redhat.com


git clone https://github.com/openshift/origin.git
cd origin/examples/prometheus

oc create -f node-exporter.yaml -n kube-system
oc adm policy add-scc-to-user -z prometheus-node-exporter -n kube-system hostaccess
oc annotate ns kube-system openshift.io/node-selector= --overwrite

oc new-app -f prometheus.yaml
