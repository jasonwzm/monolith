# Monolith is Dead. Long Live the Monolith

This repository holds detailed step-by-step instructions for following along with our talk at KubeCon China 2020.  These instructions will guide you through a setup to integrate your monolith application into an Istio service mesh, highlighting advantages for release cadence, observability, and security.

## Prerequisites

We will be using a monolith application built from the [Hipster shop demo app](https://github.com/GoogleCloudPlatform/microservices-demo).

To run your monolith, you need a linux machine with docker and docker-compose install.  We're going to use the host network for this monolith, to keep things efficient and simple, but we'll need to add the following service names to `/etc/hosts`:

```bash
127.0.0.1  adservice
127.0.0.1  cartservice
127.0.0.1  checkoutservice
127.0.0.1  currencyservice
127.0.0.1  emailservice
127.0.0.1  frontend
127.0.0.1  paymentservice
127.0.0.1  productcatalogservice
127.0.0.1  recommendationservice
127.0.0.1  redis-cart
127.0.0.1  shippingservice
```

Once that is complete, you can start your monolith with:

```bash
docker-compose -f docker/docker-compose.yaml up
```
you can checkout your monolithic hipster shop in action at http://frontend:8081.

We don't actually recommend running your monolithinc application using docker-compose, but this was an easy way to take a familiar microservice demo app and run it on a single machine, mimicking the behavior of a true monolith in production.

### Add the VM to the Istio Mesh

You can follow the instructions on setting up [VM mesh expansion](https://istio.io/latest/docs/setup/install/virtual-machine/) on Istio's website.  Note that these instructions will leverage kubernetes to run Istio, and you can also run Kiali, Jager, Prometheus, and other Cloud Native tools in your K8s cluster to serve your monolith.  The monolithic hipster shop, however, runs only on your virtual machine.

Add the `WorkloadEntry` configuration to add your VM to the mesh:

```yaml
apiVersion: networking.istio.io/v1alpha3
kind: WorkloadEntry
metadata:
  name: vm
spec:
  address: <vm-address>
  labels:
    app: vm
```

> **Note**: Instructions below are broken as of now. We are working on a fix. Please check back later. Sorry for the inconvenience.

### Service Mesh Telemetry

To set up Kiali as the dashboard for your telemetry, follow the [Kiali install instructions](https://istio.io/latest/docs/tasks/observability/kiali/).

You can start your local Kiali page by doing:

```bash
istioctl dashboard kiali
```

### Add a New Service

To add a new service to your mesh, you will need to add a `VirtualService`:

```yaml
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: reviewly
spec:
  hosts:
  - reviews
  http:
  - route:
    - destination:
        host: reviewly
```

### Canary Deployment of the Service

To add a new version of the service into the mesh, modify the `VirtualService` from above:

```yaml
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: reviewly
spec:
  hosts:
  - reviews
  http:
  - route:
    - destination:
        host: reviewly
        subset: v1
      weight: 50
    - destination:
        host: reviewly
        subset: v2
      weight: 50
```

### Inject Fault into a Service

To mimic fault on one of versions of the service, modify the `VirtualService`:

```yaml
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: reviewly
spec:
  hosts:
  - reviews
  http:
  - fault:
      delay:
        fixedDelay: 7s
        percentage:
          value: 100
    route:
    - destination:
        host: reviewly
        subset: v1
      weight: 50
  - route:
    - destination:
        host: reviewly
        subset: v2
      weight: 50
```

### Enable global mTLS

Use `PeerAuthentication` to enable global mTLS:

```yaml
apiVersion: "security.istio.io/v1beta1"
kind: "PeerAuthentication"
metadata:
  name: "default"
  namespace: "istio-system"
spec:
  mtls:
    mode: STRICT
```

### Allow only GET on the Service

Use `AuthorizationPolicy` to enable access control on the Reviewly:

```yaml
apiVersion: "security.istio.io/v1beta1"
kind: "AuthorizationPolicy"
metadata:
  name: "reviewly-reader"
  namespace: default
spec:
  selector:
    matchLabels:
      app: reviewly
  rules:
  - from:
    - source:
        principals: ["cluster.local/ns/default/sa/hipster-shop"]
    to:
    - operation:
        methods: ["GET"]
```
