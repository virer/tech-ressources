
# RH ex280 exam tips

## During the exam:
- You need to be able to create and configure users from htpasswd
- You need to be able to create secrets from file
- Be able to use the value from a secret and use it for a deployments as environment variable(retain the structure)
- You need to be able to manage node label and taints(important)
- You need to be able to deploy (default) quota for new projects
- Be able to create a self signed certificate WITH specific subject
- Please connect with each users you created, it will create identities inside OCP

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
oc adm taint nodes <node fqdn> <taint name>
  
## Remove taints
oc adm taint nodes <node fqdn> <taint name>-

## Other tips
htpasswd command is part of httpd-tools package
man req
openssl req -x509 -newkey rsa:4096 -subj '/C=BE/O=GOV/OU=Wallonia/CN=example/' -nodes -keyout key.pem -out cert.pem
 
  
---

# Here below some commands that are not exam related, but good ressources :

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
