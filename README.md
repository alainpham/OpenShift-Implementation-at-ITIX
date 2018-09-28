# OpenShift-Lab
This project is my Ansible Playbook to install OpenShift on my personal server.

More to see there... Well, maybe... 



TODO : 

git clone .. 
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
ansible-playbook playbooks/deploy-prometheus.yml  -i prod.hosts

