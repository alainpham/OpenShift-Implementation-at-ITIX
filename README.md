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

4. Add Cluster admin role to your user

```
oc adm policy add-cluster-role-to-user cluster-admin YOURUSER@mail.com
```

5. Install prometheus

```
ansible-playbook openshift-ansible/playbooks/openshift-prometheus/config.yml -i prod.logging.hosts
```

Change the config map to scrape also applications

```
oc project openshift-metrics
oc create configmap prometheus  --from-file extra/prometheus/config-map/ -o yaml --dry-run | oc replace -f -
oc delete pod prometheus-0
```

6. Install Grafana
```
cd extra/grafana
./setup-grafana.sh -n prom -a -e
```

7. Install EFK

```
ansible-playbook openshift-ansible/playbooks/openshift-logging/config.yml -i prod.logging.hosts
```

8. Add Fuse stuff

```
oc login -u system:admin

BASEURL=https://raw.githubusercontent.com/jboss-fuse/application-templates/application-templates-2.1.fuse-710017-redhat-00006

oc create -n openshift -f ${BASEURL}/fis-image-streams.json

for template in eap-camel-amq-template.json \
 eap-camel-cdi-template.json \
 eap-camel-cxf-jaxrs-template.json \
 eap-camel-cxf-jaxws-template.json \
 eap-camel-jpa-template.json \
 karaf-camel-amq-template.json \
 karaf-camel-log-template.json \
 karaf-camel-rest-sql-template.json \
 karaf-cxf-rest-template.json \
 spring-boot-camel-amq-template.json \
 spring-boot-camel-config-template.json \
 spring-boot-camel-drools-template.json \
 spring-boot-camel-infinispan-template.json \
 spring-boot-camel-rest-sql-template.json \
 spring-boot-camel-teiid-template.json \
 spring-boot-camel-template.json \
 spring-boot-camel-xa-template.json \
 spring-boot-camel-xml-template.json \
 spring-boot-cxf-jaxrs-template.json \
 spring-boot-cxf-jaxws-template.json ;
 do
 oc create -n openshift -f \
 https://raw.githubusercontent.com/jboss-fuse/application-templates/application-templates-2.1.fuse-710017-redhat-00006/quickstarts/${template}
 done


oc create -n openshift -f https://raw.githubusercontent.com/jboss-fuse/application-templates/application-templates-2.1.fuse-710017-redhat-00006/fis-console-cluster-template.json

oc create -n openshift -f https://raw.githubusercontent.com/jboss-fuse/application-templates/application-templates-2.1.fuse-710017-redhat-00006/fis-console-namespace-template.json
```

9. Install Fuse Online

```
bash extra/fuse-online/install_ocp.sh --setup
bash extra/fuse-online/install_ocp.sh --project fuse-online
```

10. Install AMQ7 stuff

```
oc project openshift

oc replace --force  -f \
https://raw.githubusercontent.com/jboss-container-images/jboss-amq-7-broker-openshift-image/72-1.0.GA/amq-broker-7-image-streams.yaml

oc replace --force -f \
https://raw.githubusercontent.com/jboss-container-images/jboss-amq-7-broker-openshift-image/72-1.0.GA/amq-broker-7-scaledown-controller-image-streams.yaml


oc import-image amq-broker-72-openshift:1.0

oc import-image amq-broker-72-scaledown-controller-openshift:1.0
```

```
for template in amq-broker-72-basic.yaml \
amq-broker-72-ssl.yaml \
amq-broker-72-custom.yaml \
amq-broker-72-persistence.yaml \
amq-broker-72-persistence-ssl.yaml \
amq-broker-72-persistence-clustered.yaml \
amq-broker-72-persistence-clustered-ssl.yaml;
 do
 oc replace --force -f \
https://raw.githubusercontent.com/jboss-container-images/jboss-amq-7-broker-openshift-image/72-1.0.GA/templates/${template}
 done
```

11. Install Kafka stuff

```
oc new-project amq-streams
oc apply -f install/cluster-operator -n amq-streams
oc apply -f examples/kafka/kafka-persistent.yaml
```

