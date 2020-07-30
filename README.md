# Monolith is Dead. Long Live the Monolith

In the instructions here,  we will guide you through a setup on how you can integrate your monolith application into an Istio service mesh.

## Prerequisites

To run your monolith, you need a linux machine with docker and docker-compose install.  We're going to use the host network for this monolith, to keep things efficient and simple, but we'll need to add the following service names to `/etc/hosts`:

```
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

```
docker-compose -f docker/docker-compose.yaml up
```

We don't actually recommend running your monolithinc application using docker-compose, but this was an easy way to take a familiar microservice demo app and run it on a single machine, mimicking the behavior of a true monolith in production.

## Application

We will be using a monolith application built from the [Hipster shop demo app](https://github.com/GoogleCloudPlatform/microservices-demo).

### Add the VM to the Istio Mesh
