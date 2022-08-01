# TAP Supply Chain With Full GitOps support

## Why do this

While multi cluster TAP is great, currently the initial creation of a deliverable in a runtime cluster is not a great UX.

Today, in TAP the suggested approach is to get the deliverable object generated in the runtime cluster (that cant succeed as no matching ClusterDelivery is present), put the YAML of that object in a file, remove the status section and owner reference sections, and then apply the file to your runtime clusters.

## Approaches to make this better
There are mutliple ways to make this a smoother UX:
1. Use the kubectl plugin called "kubectl-neat" to clean up the manifest automatically and pipe its output to the runtime cluster
2. Create a supply chain (like in this repo) to push the deliverable manifest to a git repo from which either manually or preferably via a GitOps mechanism it cxan be applied to our runtime clusters.

## Using the kubectl neat plugin
While this repo is focused on the second solution propsed above, the first method is also valid and may be easier to get started with.

You can install the kubectl neat plugin using the kubectl plugin manager called [krew](https://krew.sigs.k8s.io/docs/user-guide/setup/install/):
```bash
kubectl krew install neat
```

Once you have the plugin installed, you can do the following:
```bash
export WORKLOAD_NAME=tanzu-java-web-app
export BUILD_CLUSTER_CONTEXT=tap-mc-build-cls-admin@tap-mc-build-cls
export RUN_CLUSTER_CONTEXT=tap-mc-run-cls-admin@tap-mc-run-cls
export WORKLOAD_NAMESPACE=demo-app
kubectl get workload $WORKLOAD_NAME -n $WORKLOAD_NAMESPACE -o yaml --context $BUILD_CLUSTER_CONTEXT | kubectl neat | kubectl apply -f - --context $RUN_CLUSTER_CONTEXT
```

## Using a full gitops solution with a custom supply chain
## Requirments

* this has only been tested in internet accessible environments. in Air Gapped deployments, your mileage may vary
* this has only been tested with the GitOps supply chain mechanism. With RegistryOps, your mileage may vary

## How to do this
### Create the custom Cartographer cluster config template and cluster template

The next step is to create the cartographer resources to allow us to push the deliverable manifest to git, in our platform.

```bash
kubectl apply -f config-writer.yaml
kubectl apply -f delivery-cluster-config-template.yaml
```

### Creating the custom supply chain
The first step is to fill in the values.yaml file in this repo based on your environment. These can be the same values as you have supplied for the ootb_supply_chain section in your tap installation values.

Once you have set the values in the file, we can now render and apply the supply chain manifest:
```bash
ytt -f values.yaml -f supply-chain.yaml | kubectl apply -f -
```

### Testing out the new supply chain
To test out the new supply chain, we can use the example workload in this repo
```bash
tanzu apps workload apply -f example/workload.yaml
```