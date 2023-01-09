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

# Notes

1. Run the Job with these parameters: https://tap-test-bed.svc.eng.vmware.com/job/create-tap-testbed/3378/
2. Install docker cli, kubernetes cli, azure cli, tanzu cli (with the apps plugin), tilt cli
3. [kubectx + kubens](https://github.com/ahmetb/kubectx), "a kubernetes UI app" are highly recommended

Cluster: tap-aks-jivanov
Workload: tanzu-java-web-app
Docker repo: docker.io/jonatanivanov

http://tap-gui.tap.tap-aks-jivanov.tapdemo.vmware.com
http://spring-petclinic.my-apps.tap.tap-aks-jivanov.tapdemo.vmware.com
http://tanzu-java-web-app.my-apps.tap.tap-aks-jivanov.tapdemo.vmware.com

Useful commands:
```
kubectl get workloads.carto.run
kubectl get httpproxy
kubectl port-forward services/tanzu-java-web-app-00001-private 8888:80
```
