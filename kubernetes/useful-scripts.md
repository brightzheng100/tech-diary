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

