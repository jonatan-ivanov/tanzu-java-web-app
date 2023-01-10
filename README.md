# tanzu-java-web-app

This is a sample of a Java Spring app that works with Tilt and the Tanzu Application Platform.

## Dependencies
1. [kubectl CLI](https://kubernetes.io/docs/tasks/tools/)
2. [Tilt version >= v0.23.2](https://docs.tilt.dev/install.html)
3. Tanzu CLI and the apps plugin v0.2.0 which are provided as part of [Tanzu Application Platform](https://network.tanzu.vmware.com/products/tanzu-application-platform)
4. A cluster with Tanzu Application Platform, and the "Default Supply Chain", plus its dependencies. This supply chains is part of [Tanzu Application Platform](https://network.tanzu.vmware.com/products/tanzu-application-platform).

## Application Live View
The workload is set up by default to autoconfigure the actuators. This results in that the Spring Actuators are available at TCP port 8081 and will be used by Application Live View.
Application Live View allows you see all health metrics in the TAP GUI. If you would like to have the Actuators available at TCP port 8080 you can set the
annotation `apps.tanzu.vmware.com/auto-configure-actuators` to `false`.

## Running the sample

Start the app deployment by running:

```
tilt up
```

You can hit the spacebar to open the UI in a browser. 

- > If you see an "Update error" message like the one below, then just follow the instructions and allow that context:
    ```
    Stop! tap-beta2 might be production.
    If you're sure you want to deploy there, add:
        allow_k8s_contexts('tap-beta2')
    to your Tiltfile. Otherwise, switch k8s contexts and restart Tilt.
    ```

# Notes for my cluster

- Cluster: tap-aks-jivanov
- Workload: tanzu-java-web-app
- Docker repo: docker.io/jonatanivanov

- http://tap-gui.tap.tap-aks-jivanov.tapdemo.vmware.com
- http://spring-petclinic.my-apps.tap.tap-aks-jivanov.tapdemo.vmware.com
- http://tanzu-java-web-app.my-apps.tap.tap-aks-jivanov.tapdemo.vmware.com

Useful commands:
```
kubectl get workloads.carto.run
kubectl get httpproxy
kubectl port-forward services/tanzu-java-web-app-00001-private 8888:80
```

# Walkthrough
- Run the Job with these parameters: https://tap-test-bed.svc.eng.vmware.com/job/create-tap-testbed/3378/
- Install docker cli, kubernetes cli, [azure cli](https://learn.microsoft.com/en-us/cli/azure/), tanzu cli (with the apps plugin, min 0.10.0), [tilt cli](https://github.com/tilt-dev/tilt)
- [kubectx + kubens](https://github.com/ahmetb/kubectx), "a kubernetes UI app" are highly recommended
- Install Tanzu CLI (with the apps plugin): https://docs.vmware.com/en/VMware-Tanzu-Application-Platform/1.3/tap/GUID-install-tanzu-cli.html (make sure you download the version that matches to your TAP version, see in the jobs parameters)
- Follow the instructions in the email
  - You don't need to pass your creds to `az login`, just run `az login`, it'll open your browser
  - If you get an exception during kube ctx set, you might need to remove the kuberneres-cli config files in your `$HOME`
- Set the namespace to `my-apps` (you can use `kubens` or add the namespace to the `kubectl` command as an argument) and run `kubectl get pods` you should see petclinic pods, one of them should be running (`petclinic-db-...` is the actual app)
- Run `kubectl get httpproxy -A`, it should give you the hostname for petclinic, the url should look like this:
`http://spring-petclinic.my-apps.tap.<CLUSTER-NAME>.tapdemo.vmware.com`
WARNING: you need to use a browser that will not upgrade your connection to HTTPS (HTTPS gives you connection reset) (e.g.: Chrome)
- `kubectl get httpproxy -A` should also give you the hostname for the TAP GUI, the url should look like this:
`http://tap-gui.tap.<CLUSTER-NAME>.tapdemo.vmware.com`
WARNING: open it with a browser that lets you ignore TLS certificate errors (e.g.: Chrome)
- Click on the add `(+)` button on he left, you should see the list of accelerators
- Click on Tanzu Java Web App (not the "Restful" one)
  - Set the name for the project
  - Set the repository to `docker.io/<DOCKER-USERNAME>`
  - Next, next, download, extract, open it with a text editor
- Open `config/workload.yaml` with your editor
- Add `apps.tanzu.vmware.com/has-tests: "true"` to `metadata.labels`, remove the whole `spec.source` block and add a `spec.build` block, it should look like this:
```
apiVersion: carto.run/v1alpha1
kind: Workload
metadata:
  name: tanzu-java-web-app
  labels:
    apps.tanzu.vmware.com/workload-type: web
    apps.tanzu.vmware.com/has-tests: "true"
    apps.tanzu.vmware.com/auto-configure-actuators: "true"
    app.kubernetes.io/part-of: tanzu-java-web-app
spec:
  params:
  - name: annotations
    value:
      autoscaling.knative.dev/minScale: "1"
  build:
    env:
    - name: BP_JVM_VERSION
      value: "17.*"
```
- Open `Tiltfile`
  - Add `allow_k8s_contexts('<CONTEXT-NAME>')` to the bottom of the file (the context name is in the email or run `kubectl config current-context` to get it)
  - Modify the default value of the `NAMESPACE` variable: `NAMESPACE = os.getenv("NAMESPACE", default='my-apps')`
- Run `docker login`
- Run `tilt up` (hit space to open your browser to see the output) and wait for it to finish (it will push your source code (!) into a repo under your DockerHub account)
- Run `kubectl get httpproxy -A`, it should give you the hostname for your app, the url should look like this:
`http://<APP-NAME>.my-apps.tap.<CLUSTER-NAME>.tapdemo.vmware.com`
WARNING: you need to use a browser that will not upgrade your connection to HTTPS (HTTPS gives you connection reset)
