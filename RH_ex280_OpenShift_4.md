
# RH ex280 exam tips

## During the exam:
- You need to be able to create and configure users using htpasswd tool
- You need to be able to create and update users password secrets from htpasswd file
- Be able to create and manage groups
- Configure and understand users role (project admin, basic-user vs self-provisionner for example)
- You need to be able to create secrets from file
- Be able to use the value from a secret and use it for a deployments as environment variable(retain the structure)
- You need to be able to manage node label and taints(important)
- Be able to change SCC of a normal service account to "anyuid"
- You need to be able to deploy (default) quota for new projects
- Be able to create a self signed certificate WITH specific subject
- Be able to remove kubeadmin access
- Be able to manage ingresses and routes
- Be able to annotate route
Tips: connect with each users you created, it will create identities inside OCP

The documentation is accessible during the exam BUT the documentation format is this one (and not docs.openshift.com):
https://access.redhat.com/documentation/en-us/openshift_container_platform/4.5/

## Topics to find inside documentations:
- Be able to find HTPasswd Identity Provider inside "Authentication and authorization" section
- Be able to find limits and quota "Development" -> "Applications" > "Quota"
- Be able to find scheduler policy: "Development" -> "Nodes" > "Creating a scheduler policy file"
- Be able to find "Machine set" : "Management" -> "Machine Management"

## Using secret value as environment variable
 	apiVersion: v1
 	kind: Pod
 	metadata:
 	  name: secret-env-pod
 	spec:
 	  containers:
 	  - name: mycontainer
          image: redis
	    env:
	      - name: SECRET_USERNAME
	        valueFrom:
	          secretKeyRef:
	            name: mysecret
	            key: username
	      - name: SECRET_PASSWORD
	        valueFrom:
	          secretKeyRef:
	            name: mysecret
	            key: password

## Add taints
`oc adm taint nodes <node fqdn> <taint name>`
  
## Remove taints
`oc adm taint nodes <node fqdn> <taint name>-`

## Command list
`
oc --help   
oc adm --help  
oc adm policy --help  
oc adm options  
	
oc login  
oc whoami  
	
oc new-project my-new-project-name  
oc new-app https://github.com/...  
oc get pods -o wide -n my-namespace  
oc logs my-deployments  
oc get projects  
oc projects  
oc project my-existing-project  
oc delete project my-existing-project  
oc get -o yaml <resource>  
oc get -o json <resource>  
oc create -f resource.yaml  
oc create -f resource.json  
oc apply -f resource.yaml  
oc replace -f resource.yaml  
oc get secret -n openshift-config  
oc extract secret/secretname  
oc create secret generic MYSecretName --from-file=/tmp/filename --dry-run -o yaml -n MyNameSpace | oc replace -f  
oc rsync ...  
oc rsh ...  
  
oc status  
oc get co  
oc get <operator name>  
oc describe <operator name>  
	  
oc api-resources  
oc get endpoints  
oc get services  
oc get route  
oc expose service <service_name> [--port=1234] [--protocol=<PROTOCOL>]  
oc create route edge --service=<service_name> --cert=public.pem --key=private.pem --hostname=my.hostname.domain.tld  
  
oc get quota -n <project-name>  
oc create quota <quota-name> --hard=count/<resource>.<group>=<quota>,count/<resource>.<group>=<quota>  
oc describe quota -n <project-name>  
  
oc get users  
oc delete user/<username> # also delete it from IDP (see below)   
oc get identify  
oc delete "identity/IDP_NAME:<username>"  
oc adm groups new <groupname>  
oc adm groups add-users <groupname> <username1> [<username2>]  
oc adm groups remove-users <groupname> <username1> [<username2>]  
oc adm policy add-role-to-user <role-name> <username> -n <project-name>  
oc adm policy add-role-to-user system:image-puller system:serviceaccount:ProjectName-1:serviceAccountName: -n MyOtherProjectName  
oc adm policy add-role-to-group <role-name> <groupname> -n <project-name>  
oc adm policy add-cluster-role-to-user cluster-admin <username>   
oc get sa  
oc create sa MyServiceAccountName  
oc describe sa/MyServiceAccountName  
oc create role <role-name> --verb=<(get,list,patch,...)> --resource=<resource> -n <project-name>  
oc create clusterrole <role-name> --verb=<(get,list,patch,...)> --resource=<resource>   
oc delete secrets kubeadmi -n kube-system # This remove the kubeadmin access  
	  
oc logs --version=1 dc/deploymentconfig-name  
oc logs -f pod/mypod -c <container_name> # for multi containers pod  
  
oc scale --replicas=18 dc/deploymentConfigName  
oc autoscale dc/deploymentConfigName --min 2 --max 8 --cpu-percent=70  
	
oc get machinesets -n openshift-machine-api  
oc scale --replicas=2 machineset <MyMachineSet> -n openshift-machine-api  
oc edit machineset <MyMachineSet> -n openshift-machine-api  
oc edit MachineAutoscaler -n openshift-machine-api  
`
	  
## Other tips
htpasswd command is part of httpd-tools package  
man req  
openssl req -x509 -newkey rsa:4096 -subj '/C=BE/O=GOV/OU=Wallonia/CN=example/' -nodes -keyout key.pem -out cert.pem  
 
  
---

# Below commands not exam related, but good ressources :

## Change deployments strategy
oc patch deployments/mydeployment --patch '{"spec":{"strategy":{"type":"RollingUpdate"}}}'

## Restart deployments
oc rollout restart deployments/mydeployment

## Get service account token inside container
TOKEN=$( cat /run/secrets/kubernetes.io/serviceaccount/token ) 

---

## Confirm the service is up:
oc get svc image-registry -n openshift-image-registry
## Create a ServiceAccount:
oc create sa pipeline
## Add image-builder role to the ServiceAccount:
oc adm policy add-role-to-user system:image-builder -z pipeline
## Add privileged Security Context Constraint (SCC) so you can run container inside container:
oc adm policy add-scc-to-user privileged -z pipeline
## Set the ServiceAccount token:
TOKEN=$(oc get secret $(oc get secret | grep pipeline-token | head -1 | awk '{print $1}') -o jsonpath="{.data.token}" | base64 -d -w0)
oc create secret generic pipeline-sa-token --from-literal='token'=${TOKEN}

_Source : https://gist.github.com/luckylittle -> registry_access.sh

---

# How can I list all resources and custom resources in OpenShift
## List all CRDs with CR name and Scope
`oc get crd -o=custom-columns=NAME:.metadata.name,CR_NAME:.spec.names.singular,SCOPE:.spec.scope`

## List every single custom resources in the cluster
`oc get $(oc get crd -o=custom-columns=CR_NAME:.spec.names.singular --no-headers | awk '{printf "%s%s",sep,$0; sep=","}') --ignore-not-found --all-namespaces -o=custom-columns=KIND:.kind,NAME:.metadata.name,NAMESPACE:.metadata.namespace`

## List every single resource in the cluster (custom and non-custom)
`oc get $(oc api-resources --verbs=list -o name | awk '{printf "%s%s",sep,$0;sep=","}')  --ignore-not-found --all-namespaces -o=custom-columns=KIND:.kind,NAME:.metadata.name,NAMESPACE:.metadata.namespace --sort-by='metadata.namespace'`

## List every single non-namespaced resource in the cluster
`oc get $(oc api-resources --namespaced=false --verbs=list -o name | awk '{printf "%s%s",sep,$0;sep=","}')  --ignore-not-found --all-namespaces -o=custom-columns=KIND:.kind,NAME:.metadata.name --sort-by='kind'`

## List every single namespaced resource in a namespace
`oc get $(oc api-resources --namespaced=true --verbs=list -o name | awk '{printf "%s%s",sep,$0;sep=","}')  --ignore-not-found -n ${NAMESPACE} -o=custom-columns=KIND:.kind,NAME:.metadata.name --sort-by='kind'`

_Note: `oc get all` inside namespace does not really show "all" and the above commands will fill those gaps._

_Source: https://gist.github.com/luckylittle -> ocp4_all_resources.md
