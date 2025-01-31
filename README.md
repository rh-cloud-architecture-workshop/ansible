# Deployment of operators and other assets, and setup users for ARO


## Setup CNA 
ansible-playbook playbooks/ocp4_workload_cloud_architecture_workshop.yml -e ACTION=create


## create users

htpasswd -c -B aro-users.db admin
htpasswd -B aro-users.db user1
htpasswd -B aro-users.db user2


oc create secret generic htp-secret --from-file ./aro-users.db -n openshift-config
oc adm policy add-cluster-role-to-user cluster-admin admin --rolebinding-name=cluster-admin

oc get oauth cluster -o yaml > aro-oauth.yaml  

vi aro-oauth.yaml

#replace spec: {} with the following
spec:
  identityProviders:
  - htpasswd:
      fileData:
        name: htp-secret
    mappingMethod: claim
    name: aro-users
    type: HTPasswd


oc replace -f ./aro-oauth.yaml

oc create secret generic htp-secret --from-file htpasswd=./aro-users.db --dry-run=client -o yaml | oc replace -n openshift-config -f -
