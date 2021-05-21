---
title: "What the heck is a data plane?"
date: 2021-05-12T12:49:07+01:00
draft: false
tags: [service mesh , data plane]
---

![Paper Airplane Sets Record With 82-Mile Flight](https://www.treehugger.com/thmb/zHEqqllMa66MSzzitQ8G5pp136Y=/768x0/filters:no_upscale():max_bytes(150000):strip_icc()/__opt__aboutcom__coeus__resources__content_migration__mnn__images__2014__09__shutterstock_556793080-b4a5fd4b2287434b959ef955e39a7aa7.jpg)

As i sit here at 12:50am . I attempt to answer the question that's been on my mind for days now.  "What the heck is a data plane?". 

To fully understand we need to back track a bit. A data plane is one of two major components of a [service mesh](https://www.redhat.com/en/topics/microservices/what-is-a-service-mesh) ,  yet that alone is far from the answer i seek.


## Taking a closer look

To find the answers i seek, it's worth taking look at the architecture of a typical service mesh and analyze from there. 

![istio architecture](https://istio.io/latest/docs/ops/deployment/architecture/arch.svg "istio service mesh architecture")


When a request comes into service A on lets  `/login` the data plane ( AKA the sidecar proxy, in this case the envoy instance in the diagram ) is responsible for resolving what service the request is to be forwarded to, collect metrics , perform healthchecks and a bunch of other stuff the service mesh user might have configred it to do. 


## TL;DR

A data plane is a component of a service mesh responsible for:

- Routing
- Load Balancing 
- Health Checks
- Observability 
- Service Discovery 

Popular examples inclued , [Envoy proxy](https://envoyproxy.io), [Nginx](https://nginx.com) , [Traefik](https://traefik.io/).


				



