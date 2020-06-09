# FAQs

## How-tos

### How to force delete a namespace?

Sometimes a namespace will be pending at `Terminating` state forever for some reason. So here are the simple steps to force delete it:

```text
export NAMESPACE_TO_DELETE=app-entitlement
kubectl get ns $NAMESPACE_TO_DELETE -o json | jq '.spec.finalizers=[]' > ns-without-finalizers.json
kubectl replace --raw "/api/v1/namespaces/$NAMESPACE_TO_DELETE/finalize" -f ./ns-without-finalizers.json
rm -f ns-without-finalizers.json
```

### How to patch service account with image pull secret?

```text
kubectl patch serviceaccount default \
    -p '{"imagePullSecrets": [{"name": "multiclusterhub-operator-pull-secret"}]}' \
    -n multicluster-endpoint
```



