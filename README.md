# local-openfaas-workbench

This repository contains my notes for how to quickly provision a local environment for tinkering with OpenFaaS.

## Prerequisite Steps:

1. `brew update && brew install helm kind helmfile faas-cli`
2. `helm plugin install https://github.com/databus23/helm-diff --version v3.0.0-rc.7`
   * at the time of writing this, there wasn't a stable release of the diff plugin, but this required to avoid errors with helm3 and using helmfile

## Provision The Cluster And Install OpenFaaS

1. `kind create cluster`
2. Ensure you're using the kind cluster context with kubectl: `kubectl config set current-context kind-kind`
3. Verify the cluster is working: `kubectl get nodes`
4. Install OpenFaaS to the local kind cluster: `helmfile apply`
5. Show the state of the installed chart: `helm list -n openfaas`
6. Wait for OpenFaaS pods to become ready: `kubectl get po -n openfaas --watch`
7. Add `127.0.0.1 gateway.openfaas.local` to `/etc/hosts` to map the [default host entry](https://github.com/openfaas/faas-netes/blob/master/chart/openfaas/values.yaml#L118) for OpenFaaS ingress
8. Open http://gateway.openfaas.local:30080/ui/ to verify you can see the OpenFaaS UI

## Create a hello world function to verify everything is working correctly

This does nothing more than download the community template for node10 functions, which uses express under the covers.

1. Pull the community node10 template: `faas-cli template store pull node10-express`
   * according to the OpenFaaS docs, this leverages a more performant implementation for functions
2. Use the template to scaffold a function: `faas-cli new --lang node10-express hello-world --gateway http://gateway.openfaas.local:30080`
3. Build the docker image for the function: `faas-cli build -f hello-world.yml`
4. Load the image into kind so it can be deployed: `kind load docker-image hello-world:latest`
5. Deploy the function: `faas-cli up --skip-push -n openfaas-fn --update -f hello-world.yml --gateway http://gateway.openfaas.local:30080`
6. You can now interact with the function with the OpenFaaS UI at http://gateway.openfaas.local:30080/ui/

## Wipe It Out

When you're done, you can wipe the slate clean by running:

* `kind delete cluster`

## TODO

* [ ] Investigate using knative build with OpenFaaS
  * https://blog.alexellis.io/first-look-at-knative-build-for-openfaas-functions/
* [x] Investigate kind configuration that allows for a NodePort mapping to avoid having to run port forwarding commands
  * https://kind.sigs.k8s.io/docs/user/quick-start#mapping-ports-to-the-host-machine
  * Chose nginx for now, since I'm familiar with that
  * Spent time on [Contour](https://projectcontour.io), which looks promising, but would require more work to write ingress config via their CRDs
  * Briefly looked at [Gloo Ingress](https://docs.solo.io/gloo/latest/installation/ingress/), but their helm chart leaves a lot to be desired
* [ ] Use a private local registry to avoid having to do a `kind load foo` each time the function is rebuilt
  * https://kind.sigs.k8s.io/docs/user/private-registries/
* [ ] Create a node 12 hapi-based template, possibly submit it to OpenFaaS
* [ ] Add OIDC-based authentication

## References

* https://blog.alexellis.io/get-started-with-openfaas-and-kind/
* https://projectcontour.io/kindly-running-contour/