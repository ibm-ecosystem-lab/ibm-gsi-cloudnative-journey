---
title: Services & Networking
---
<!--- cSpell:ignore ICPA openshiftconsole Theia userid toolset crwexposeservice gradlew bluemix ocinstall Mico crwopenlink crwopenapp swaggerui gitpat gituser  buildconfig yourproject wireframe devenvsetup viewapp crwopenlink  atemplatized rtifactoryurlsetup Kata Koda configmap Katacoda checksetup cndp katacoda checksetup Linespace igccli regcred REPLACEME Tavis pipelinerun openshiftcluster invokecloudshell cloudnative sampleapp bwoolf hotspots multicloud pipelinerun Sricharan taskrun Vadapalli Rossel REPLACEME cloudnativesampleapp artifactoryuntar untar Hotspot devtoolsservices Piyum Zonooz Farr Kamal Arora Laszewski  Roadmap roadmap Istio Packt buildpacks automatable ksonnet jsonnet targetport podsiks SIGTERM SIGKILL minikube apiserver multitenant kubelet multizone Burstable checksetup handson  stockbffnode codepatterns devenvsetup newwindow preconfigured cloudantcredentials apikey Indexyaml classname  errorcondition tektonpipeline gradlew gitsecret viewapp cloudantgitpodscreen crwopenlink cdply crwopenapp -->

## Services

An abstract way to expose an application running on a set of Pods as a network service.

Kubernetes Pods are mortal. They are born and when they die, they are not resurrected. If you use a Deployment to run your app, it can create and destroy Pods dynamically.

Each Pod gets its own IP address, however in a Deployment, the set of Pods running in one moment in time could be different from the set of Pods running that application a moment later.

In Kubernetes, a Service is an abstraction which defines a logical set of Pods and a policy by which to access them (sometimes this pattern is called a micro-service). The set of Pods targeted by a Service is usually determined by a selector (see below for why you might want a Service without a selector).

If you’re able to use Kubernetes APIs for service discovery in your application, you can query the API server for Endpoints, that get updated whenever the set of Pods in a Service changes.

For non-native applications, Kubernetes offers ways to place a network port or load balancer in between your application and the backend Pods.


# Resources

**IKS & OpenShift**
- [Services](https://kubernetes.io/docs/concepts/services-networking/service/)
- [Exposing Services](https://kubernetes.io/docs/tutorials/kubernetes-basics/expose/expose-intro/)

# References

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-deployment
  labels:
    app: nginx
    version: v1
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
        version: v1
    spec:
      containers:
      - name: nginx
        image: bitnami/nginx
        ports:
        - containerPort: 8080
          name: http
---
apiVersion: v1
kind: Service
metadata:
  name: my-service
spec:
  selector:
    app: nginx
  ports:
    - name: http
      port: 80
      targetPort: http
```

=== "OpenShift"

    ** Get Service **
    ```
     oc get svc
    ```
    ** Get Service Description **
    ```
    oc describe svc my-service
    ```
    ** Expose a service **
    ```
    oc expose service <service_name>
    ```
    ** Get Route for the Service **
    ```  
    oc get route
    ```


=== "IKS"

    ** Get Service **
    ```
    kubectl get svc
    ```
    ** Get Service Description **
    ```
    kubectl describe svc my-service
    ```
    ** Get Service Endpoints **
    ```
    kubectl get ep my-service
    ```
    ** Expose a Deployment via a Service **
    ```
     kubectl expose deployment my-deployment --port 80 --target-port=http --selector app=nginx --name my-service-2 --type NodePort
    ```


# Routes

(OpenShift Only)

Routes are Openshift objects that expose services for external clients to reach them by name.  

Routes can be insecure or secured on creation using certificates.

The new route inherits the name from the service unless you specify one using the --name option.

# Resources

**OpenShift**
- [Routes](https://docs.openshift.com/online/pro/dev_guide/routes.html)
- [Route Configuration](https://docs.openshift.com/container-platform/4.3/networking/routes/route-configuration.html)
- [Secured Routes](https://docs.openshift.com/container-platform/4.3/networking/routes/secured-routes.html)

# References

** Route Creation **
```
apiVersion: v1
kind: Route
metadata:
  name: frontend
spec:
  to:
    kind: Service
    name: frontend
```
** Secured Route Creation **
```
apiVersion: v1
kind: Route
metadata:
  name: frontend
spec:
  to:
    kind: Service
    name: frontend
  tls:
    termination: edge
```

### Commands

=== "OpenShift"

    ** Create Route from YAML **
    ```
    oc apply -f route.yaml
    ```
    ** Get Route **
    ```
    oc get route
    ```
    ** Describe Route **
    ```
    oc get route <route-name>
    ```
    ** Get Route YAML **
    ```
    oc get route <route-name> -o yaml
    ```



## Ingress

An API object that manages external access to the services in a cluster, typically HTTP.

Ingress can provide load balancing, SSL termination and name-based virtual hosting.

Ingress exposes HTTP and HTTPS routes from outside the cluster to services within the cluster. Traffic routing is controlled by rules defined on the Ingress resource.

# Resources

**OpenShift**
- [Ingress Operator](https://docs.openshift.com/container-platform/4.3/networking/ingress-operator.html)
- [Using Ingress Controllers](https://docs.openshift.com/container-platform/4.3/networking/configuring_ingress_cluster_traffic/configuring-ingress-cluster-traffic-ingress-controller.html)

**IKS**
- [Ingress](https://kubernetes.io/docs/concepts/services-networking/ingress/)
- [Ingress Controllers](https://kubernetes.io/docs/concepts/services-networking/ingress-controllers/)
- [Minikube Ingress](https://kubernetes.io/docs/tasks/access-application-cluster/ingress-minikube/)

# References

```yaml
apiVersion: networking.k8s.io/v1beta1 # for versions before 1.14 use extensions/v1beta1
kind: Ingress
metadata:
  name: example-ingress
spec:
  rules:
  - host: hello-world.info
    http:
      paths:
      - path: /
        backend:
          serviceName: web
          servicePort: 8080
```

=== "Openshift"

    ** View Ingress Status **
    ```
     oc describe clusteroperators/ingress
    ```
    ** Describe default Ingress Controller **
    ```
     oc describe --namespace=openshift-ingress-operator ingresscontroller/default
    ```



=== "IKS"

    ```
    minikube addons enable ingress
    ```
    ```
    kubectl get pods -n kube-system | grep ingress
    ```
    ```
    kubectl run web --image=bitnami/nginx --port=8080
    ```
    ```
    kubectl expose deployment web --target-port=8080 --type=NodePort
    ```
    ```
    kubectl get svc web
    ```
    ```
    minikube service --url web
    ```

    ```
    stern ingress -n kube-system
    ```
    ```
    kubectl get ingress
    ```
    ```
    kubcetl describe ingress example-ingress
    ```
    ```
    curl hello-world.info --resolve hello-world.info:80:<ADDRESS>
    ```



## Activities

| Task                            | Description         | Link        |
| --------------------------------| ------------------  |:----------- |
| *** Try It Yourself ***                         |         |         | 
| Creating Services | Create two services with certain requirements. | [Setting up Services](../activities/labs/lab8) |