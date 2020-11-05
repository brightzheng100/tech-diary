# Useful Scripts

## cleanup\_ns.sh

To clean up all namespaced objects and eventually delete the namespace.

```text
#!/bin/bash

ns=$1
if [[ -z "$ns" ]] ; then
  echo "Usage: $0 <NAMESPACE>"
  exit 1
fi

echo "---> namespace: $ns"

# Retrieve all potential namespaced types
all_types=$(kubectl api-resources --namespaced=true --verbs=delete -o name | tr "\n" "," | sed -e "s/,$//")
#all_types="helmreleases.apps.open-cluster-management.io"

# Retrieve all potential namespaced objects
all_objects=($(kubectl get "$all_types" -n $ns -o json | jq -r '.items[] | .apiVersion + ";" + .kind + ";" + .metadata.name'))
#echo "---> all_objects: ${all_objects[@]}"

# Iterate to delete all namespaced objects
tmpfile="$(mktemp /tmp/deleting-object.XXXXXX)"
for item in "${all_objects[@]}"; do
  k8s_kind1=$(echo "$item" | cut -d ";" -f 1 | cut -d "/" -f 1)
  k8s_kind2=$(echo "$item" | cut -d ";" -f 2)
  k8s_kind="$k8s_kind2.$k8s_kind1"            # make it a full name like TagLabelMap.infra.management.ibm.com
  k8s_name=$(echo "$item" | cut -d ";" -f 3)
  echo "---> kind:$k8s_kind; name: $k8s_name";

  # Remove the finalizers
  kubectl get "$k8s_kind/$k8s_name" -n $ns -o json | jq '.metadata.finalizers=[]' > "$tmpfile"

  # Replace the object with removed finalizers
  kubectl -n $ns replace -f "$tmpfile"

  # Finally delete it
  kubectl -n $ns delete "$k8s_kind/$k8s_name"
done

# delete the ns at the end
kubectl get ns $ns -o json | jq '.spec.finalizers=[]' > "$tmpfile"
kubectl replace --raw "/api/v1/namespaces/$ns/finalize" -f "$tmpfile"
kubectl delete ns $ns

rm "$tmpfile" || true
```

## Find out the CRDs

There are some default CRDs but sometimes it's important to find out what more CRDs are added after some specific installations.

For example, we can find out what CRDs have been installed within ROKS, with OCP v4.3.38\_1544, by default:

```text
$ kg crd -o json | jq -r '.items[].metadata.name'
```

The default CRDs in ROKS include:

```text
aksclusterproviderspecs.aks.mcm.ibm.com
alertmanagers.monitoring.coreos.com
alertrules.monitoringcontroller.cloud.ibm.com
apiservers.config.openshift.io
applications.app.k8s.io
auditpolicies.audit.policies.ibm.com
authentications.config.openshift.io
authentications.operator.openshift.io
baremetalhosts.metal3.io
bgpconfigurations.crd.projectcalico.org
bgppeers.crd.projectcalico.org
blockaffinities.crd.projectcalico.org
builds.config.openshift.io
catalogsourceconfigs.operators.coreos.com
catalogsources.operators.coreos.com
certificatepolicies.policies.ibm.com
certificaterequests.certmanager.k8s.io
certificates.certmanager.k8s.io
challenges.certmanager.k8s.io
channels.app.ibm.com
clients.oidc.security.ibm.com
clusterimagepolicies.securityenforcement.admission.cloud.ibm.com
clusterinformations.crd.projectcalico.org
clusterissuers.certmanager.k8s.io
clusteroperators.config.openshift.io
clusterresourcequotas.quota.openshift.io
clusters.cluster.k8s.io
clusterservicestatuses.clusterhealth.ibm.com
clusterserviceversions.operators.coreos.com
clusterversions.config.openshift.io
compliances.compliance.mcm.ibm.com
configs.imageregistry.operator.openshift.io
configs.samples.operator.openshift.io
consoleclidownloads.console.openshift.io
consoleexternalloglinks.console.openshift.io
consolelinks.console.openshift.io
consolenotifications.console.openshift.io
consoles.config.openshift.io
consoles.operator.openshift.io
consoleyamlsamples.console.openshift.io
credentialsrequests.cloudcredential.openshift.io
deployables.app.ibm.com
dnses.config.openshift.io
dnses.operator.openshift.io
dnsrecords.ingress.operator.openshift.io
eksclusterproviderspecs.eks.mcm.ibm.com
featuregates.config.openshift.io
felixconfigurations.crd.projectcalico.org
gkeclusterproviderspecs.gke.mcm.ibm.com
globalnetworkpolicies.crd.projectcalico.org
globalnetworksets.crd.projectcalico.org
helmreleases.app.ibm.com
hostendpoints.crd.projectcalico.org
iampolicies.iam.policies.ibm.com
ibmservicesplatforms.operator.ibm.com
iksclusterproviderspecs.iks.mcm.ibm.com
imagecontentsourcepolicies.operator.openshift.io
imagepolicies.securityenforcement.admission.cloud.ibm.com
images.config.openshift.io
infrastructures.config.openshift.io
ingresscontrollers.operator.openshift.io
ingresses.config.openshift.io
installations.operator.tigera.io
installplans.operators.coreos.com
ipamblocks.crd.projectcalico.org
ipamconfigs.crd.projectcalico.org
ipamhandles.crd.projectcalico.org
ippools.crd.projectcalico.org
ippools.whereabouts.cni.cncf.io
issuers.certmanager.k8s.io
kubeapiservers.operator.openshift.io
kubecontrollermanagers.operator.openshift.io
kubeschedulers.operator.openshift.io
mcoconfigs.machineconfiguration.openshift.io
monitoringdashboards.monitoringcontroller.cloud.ibm.com
navconfigurations.foundation.ibm.com
network-attachment-definitions.k8s.cni.cncf.io
networkpolicies.crd.projectcalico.org
networks.config.openshift.io
networks.operator.openshift.io
networksets.crd.projectcalico.org
oauths.config.openshift.io
ocpclusterproviderspecs.ocp.mcm.ibm.com
openshiftapiservers.operator.openshift.io
openshiftcontrollermanagers.operator.openshift.io
operatorgroups.operators.coreos.com
operatorhubs.config.openshift.io
operatorpkis.network.operator.openshift.io
operatorsources.operators.coreos.com
orders.certmanager.k8s.io
passwordrules.icp.ibm.com
placementrules.app.ibm.com
podmonitors.monitoring.coreos.com
policies.policy.mcm.ibm.com
projects.config.openshift.io
prometheuses.monitoring.coreos.com
prometheusrules.monitoring.coreos.com
proxies.config.openshift.io
rbacsyncs.ibm.com
rolebindingrestrictions.authorization.openshift.io
schedulers.config.openshift.io
securitycontextconstraints.security.openshift.io
servicecas.operator.openshift.io
servicecatalogapiservers.operator.openshift.io
servicecatalogcontrollermanagers.operator.openshift.io
servicemonitors.monitoring.coreos.com
subscriptions.app.ibm.com
subscriptions.operators.coreos.com
tigerastatuses.operator.tigera.io
tuneds.tuned.openshift.io
```

We then can set a baseline as a file, say name `_crds_roks.txt`. After installation of RHACM, we can check what other CRDs have been added:

```text
# List all CRDs added on top of default ones from ROKS
$ kg crd -o json | jq -r '.items[].metadata.name' | grep -Ev "$(cat _roks_default_crds.txt | tr '\n' '|')"
```

Then we would get a list like:

```text
baremetalassets.inventory.open-cluster-management.io
channels.apps.open-cluster-management.io
checkpoints.hive.openshift.io
clusterdeployments.hive.openshift.io
clusterdeprovisions.hive.openshift.io
clusterimagesets.hive.openshift.io
clustermanagers.operator.open-cluster-management.io
clusterprovisions.hive.openshift.io
clusterrelocates.hive.openshift.io
clusterstates.hive.openshift.io
deployables.apps.open-cluster-management.io
dnszones.hive.openshift.io
helmreleases.apps.open-cluster-management.io
hiveconfigs.hive.openshift.io
klusterletaddonconfigs.agent.open-cluster-management.io
machinepoolnameleases.hive.openshift.io
machinepools.hive.openshift.io
managedclusteractions.action.open-cluster-management.io
managedclusterinfos.internal.open-cluster-management.io
managedclusters.cluster.open-cluster-management.io
managedclusterviews.view.open-cluster-management.io
manifestworks.work.open-cluster-management.io
multiclusterhubs.operator.open-cluster-management.io
placementbindings.policy.open-cluster-management.io
placementrules.apps.open-cluster-management.io
policies.policy.open-cluster-management.io
searchservices.search.acm.com
selectorsyncidentityproviders.hive.openshift.io
selectorsyncsets.hive.openshift.io
subscriptions.apps.open-cluster-management.io
syncidentityproviders.hive.openshift.io
syncsetinstances.hive.openshift.io
syncsets.hive.openshift.io
userpreferences.console.open-cluster-management.io
```

So if we want to check what extra CRDs are added, on top of the new baseline with ROKS + RHACM, do this:

```text
# List all CRDs added on top of default ones from ROKS + RHACM
kg crd -o json | jq -r '.items[].metadata.name' | grep -Ev "$(cat _crds_roks.txt | tr '\n' '|')""|""$(cat _crds_rhacm.txt | tr '\n' '|')"
```

