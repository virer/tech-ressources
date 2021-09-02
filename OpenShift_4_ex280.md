

# Here below some commands that are not exam related, but good ressources :

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
## Source : 
#  https://gist.github.com/luckylittle -> registry_access.sh

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
# Source : 
#  https://gist.github.com/luckylittle -> ocp4_all_resources.md
