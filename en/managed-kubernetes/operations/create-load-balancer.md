# Granting access to an app running in a {{ k8s }} cluster

To grant access to an app running in a {{ k8s }} cluster, you can use [various types of public and internal services](../concepts/service.md).

To publish an app, use a `LoadBalancer` service. The following options are supported:
* Public access by IP address with a [network load balancer](../../network-load-balancer/concepts/index.md).
* Access from internal networks by IP address with an [internal network load balancer](../../network-load-balancer/concepts/internal-load-balancer.md).
  The application will be available from [subnets](../../vpc/concepts/network.md#subnet) of {{ vpc-full-name }} or a company's internal subnets connected to {{ yandex-cloud }} through [{{ interconnect-full-name }}](../../interconnect/) or a VPN.

To use DDoS protection, [reserve](../../vpc/operations/enable-ddos-protection.md) a public IP address and [specify](#advanced) it using the `loadBalancerIP` option.

{% note info %}

Unlike the IP address of a pod or node, which may change if the resources in a node group are updated, the IP address of `LoadBalancer` services don't change.

{% endnote %}

Prepare and run the application to be granted access to using a `LoadBalancer` service in the {{ k8s }} cluster. As an example, use a simple application that responds to HTTP requests on port 8080.

* [Create a simple app](#simple-application)
* [Create a LoadBalancer service with a public IP address](#lb-create)
* [Create a LoadBalancer service with an internal IP address](#lb-int-create)
* [loadBalancerIP and externalTrafficPolicy parameters](#advanced)

## Create a simple app {#simple-app}

1. Save the following app creation specification to a YAML file named `hello.yaml`:

   [Deployment](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/) is the {{ k8s }} API object that manages the replicated application.

   ```yaml
   apiVersion: apps/v1
   kind: Deployment
   metadata:
     name: hello
   spec:
     replicas: 2
     selector:
       matchLabels:
         app: hello
     template:
       metadata:
         labels:
           app: hello
       spec:
         containers:
         - name: hello-app
           image: cr.yandex/crpjd37scfv653nl11i9/hello:1.1
   ```

1. Create an application:

   {% list tabs %}

   - CLI

     {% include [cli-install](../../_includes/cli-install.md) %}

     {% include [default-catalogue](../../_includes/default-catalogue.md) %}

     ```bash
     kubectl apply -f hello.yaml
     ```

     Command execution result:

     ```bash
     deployment.apps/hello created
     ```

   {% endlist %}

1. View information about the created app:

   {% list tabs %}

   - CLI

     ```bash
     kubectl describe deployment hello
     ```

     Command execution result:

     ```bash
     Name:                   hello
     Namespace:              default
     CreationTimestamp:      Wed, 28 Oct 2020 23:15:25 +0300
     Labels:                 <none>
     Annotations:            deployment.kubernetes.io/revision: 1
     Selector:               app=hello
     Replicas:               2 desired | 2 updated | 2 total | 1 available | 1 unavailable
     StrategyType:           RollingUpdate
     MinReadySeconds:        0
     RollingUpdateStrategy:  25% max unavailable, 25% max surge
     Pod Template:
       Labels:  app=hello
       Containers:
        hello-app:
         Image:        cr.yandex/crpab12cdef353nl11i9/hello:1.1
         Port:         <none>
         Host Port:    <none>
         Environment:  <none>
         Mounts:       <none>
       Volumes:        <none>
     Conditions:
       Type           Status  Reason
       ----           ------  ------
       Available      False   MinimumReplicasUnavailable
       Progressing    True    ReplicaSetUpdated
     OldReplicaSets:  <none>
     NewReplicaSet:   hello-74c9c1b238 (2/2 replicas created)
     Events:
       Type    Reason             Age   From                   Message
       ----    ------             ----  ----                   -------
       Normal  ScalingReplicaSet  10s   deployment-controller  Scaled up replica set hello-74c9c1b238 to 2
     ```

   {% endlist %}

## Create a LoadBalancer service with a public IP address {#lb-create}

When you create a `LoadBalancer` service, the {{ yandex-cloud }} controller creates and configures a [network load balancer](../../network-load-balancer/concepts/index.md) in your folder with a public IP address.

{% note warning %}

* You will be charged for the network load balancer you created based on the [pricing rules](../../network-load-balancer/pricing.md).
* Don't modify or delete the network load balancer or target groups that are automatically created in your folder after creating a `LoadBalancer` service.

{% endnote %}

1. Save the following specification for creating a `LoadBalancer` service to a YAML file named `load-balancer.yaml`:

   ```yaml
   apiVersion: v1
   kind: Service
   metadata:
     name: hello
   spec:
     ports:
     - port: 80
       name: plaintext
       targetPort: 8080
     selector:
       app: hello
     type: LoadBalancer
   ```

   Where:
   * `port` is the network load balancer port to handle user requests.
   * `targetPort` is the container port where the application is available.
   * `selector` stands for the selector labels used in the pod template when creating the `Deployment` object.

1. Create a network load balancer:

   {% list tabs %}

   - CLI

     ```bash
     kubectl apply -f load-balancer.yaml
     ```

     Command execution result:

     ```bash
     service/hello created
     ```

   {% endlist %}

1. View information about the network load balancer created:

   {% list tabs %}

   - Management console
     1. In the [management console]({{ link-console-main }}), select your default folder.
     1. Select **{{ network-load-balancer-name }}**.
     1. The **Load balancers** tab shows the network load balancer with the `k8s` prefix in the name and the unique ID of your {{ k8s }} cluster in the description.

   - CLI

     ```bash
     kubectl describe service hello
     ```

     Command execution result:

     ```bash
     Name:                     hello
     Namespace:                default
     Labels:                   <none>
     Annotations:              Selector:  app=hello
     Type:                     LoadBalancer
     IP:                       172.20.169.7
     LoadBalancer Ingress:     130.193.50.111
     Port:                     plaintext  80/TCP
     TargetPort:               8080/TCP
     NodePort:                 plaintext  32302/TCP
     Endpoints:                10.1.130.4:8080
     Session Affinity:         None
     External Traffic Policy:  Cluster
     Events:
       Type    Reason                Age    From                Message
       ----    ------                ----   ----                -------
       Normal  EnsuringLoadBalancer  2m43s  service-controller  Ensuring load balancer
       Normal  EnsuredLoadBalancer   2m17s  service-controller  Ensured load balancer
     ```

   {% endlist %}

1. Make sure the application is available from the internet:

   {% list tabs %}

   - CLI

     ```bash
     curl http://130.193.50.111
     ```

     Where:
     * `130.193.50.111` is the public IP address from the `LoadBalancer Ingress` field.

     Command execution result:

     ```bash
     Hello, world!
     Running in 'hello-74c9c1b238-c1rpa'
     ```

   {% endlist %}

## Create a LoadBalancer service with an internal IP address {#lb-int-create}

To create a network load balancer with an internal IP address, specify the `yandex.cloud/load-balancer-type` and `yandex.cloud/subnet-id` parameters in the YAML specification for the service under `annotations`:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: hello
  annotations:
    yandex.cloud/load-balancer-type: internal
    yandex.cloud/subnet-id: e1b23q26ab1c0dce8te9
spec:
  ports:
  - port: 80
    name: plaintext
    targetPort: 8080
  selector:
    app: hello
  type: LoadBalancer
```

Where:
* `yandex.cloud/subnet-id` is the ID of the subnet to allocate the IP address for the network load balancer in.
* `port` is the network load balancer port to handle user requests.
* `targetPort` is the container port where the application is available.
* `selector` stands for the selector labels used in the pod template when creating the `Deployment` object.

## loadBalancerIP and externalTrafficPolicy parameters {#advanced}

In {{ managed-k8s-short-name }}, the following advanced settings are available for a service with the `LoadBalancer` type:
* Assign a [pre-allocated public IP address](../../vpc/operations/get-static-ip.md) using the `loadBalancerIP` parameter.
  When reserving a static IP address, you can activate [DDoS protection](../../vpc/ddos-protection/index.md).
* Manage traffic using the [externalTrafficPolicy](https://kubernetes.io/docs/reference/kubernetes-api/service-resources/service-v1/#ServiceSpec) parameter:
  * `Cluster`: Traffic goes to any of the {{ k8s }} cluster nodes. In this case:
    * If pods are missing from the node, [kube-proxy](https://kubernetes.io/docs/reference/command-line-tools-reference/kube-proxy) forwards traffic to another node.
  * `Local`: Traffic goes directly to the nodes where the application containers are running. In this case:
    * The originating IP address of the user query is saved.
    * Horizontal traffic exchanged by VMs is lower.

The `loadBalancerIP` and `externalTrafficPolicy` parameters are optional. If you omit them, a load balancer is created with a dynamic IP address and the `externalTrafficPolicy: Cluster` parameter.

Example of a `LoadBalancer` service YAML specification with these parameters:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: hello
spec:
  ports:
  - port: 80
    name: plaintext
    targetPort: 8080
  selector:
    app: hello
  loadBalancerIP: <pre-allocated IP address>
  type: LoadBalancer
  externalTrafficPolicy: <Local or Cluster>
```