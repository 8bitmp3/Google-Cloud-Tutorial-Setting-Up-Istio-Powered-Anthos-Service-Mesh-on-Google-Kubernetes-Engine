# Setting up Istio-powered Anthos Service Mesh on Google Kubernetes Engine

This is an example of a Google Cloud [tutorial](https://github.com/8bitmp3/Google-Cloud-Tutorial-Setting-Up-Istio-Powered-Anthos-Service-Mesh-on-Google-Kubernetes-Engine/blob/master/Google-Cloud-Tutorial-Setting-Up-Istio-Powered-Anthos-Service-Mesh-on-Google-Kubernetes-Engine.md) I wrote when learning how to use Istio and Anthos.

It is designed to give a broad overview of the Anthos product suite and its major components, such as Anthos Service Mesh, Google Kubernetes Engine (GKE), and Istio for container, cluster, and service monitoring and management with Google Cloud. You will also be learning how to perform a clean installation of an Anthos Service Mesh on a Google Cloud GKE cluster in your project.

### Contents:

- Introduction
- Overview of Anthos
- Overview of an Anthos Service Mesh
    - Istio service mesh
- Kubernetes and Anthos GKE
- A brief history of containers (covering Borg, CGroups/Process Containers, Docker, and more)
- Start a new Anthos Service Mesh project
- Enable relevant APIs
- Check your IAM permissions
- Provision a new Google Kubernetes Engine cluster
- Create a service account for Istio and authenticate
- Install a Anthos Service Mesh with `istioctl`

See this [link](https://github.com/8bitmp3/Google-Cloud-Tutorial-Setting-Up-Istio-Powered-Anthos-Service-Mesh-on-Google-Kubernetes-Engine/blob/master/Google-Cloud-Tutorial-Setting-Up-Istio-Powered-Anthos-Service-Mesh-on-Google-Kubernetes-Engine.md) for the full tutorial.

---
(An extract from the tutorial)

## Overview of an Anthos Service Mesh

An Anthos Service Mesh is a network for services that manages interactions across all services. It uses a distribution of [Istio](https://istio.io/)—an open-source implementation of the service mesh infrastructure layer. 

### Istio service mesh

Istio helps configure and manage various applications and services as the mesh network scales. With Istio's mesh service platform, you can:

- Uniformly observe all their workloads and obtain automatic metrics, logs, and traces for all traffic within clusters.
- Insulate application-level code from the backends by decoupling access control systems, telemetry capturing systems, quota enforcement systems from the backends’ specific interfaces—all of that while being able to inject and control policies between application code and backend.
- Easily manage network traffic routing and splitting for microservices through decoupling traffic flow and infrastructure scaling.
- Automatically load balance for various traffic types (HTTP, gRPC, WebSocket, and TCP).
- Enforce security and encryption policy without changing applications themselves.

An Istio service mesh is logically split into a **Data Plane** and a **Control Plane**.

- The **Data Plane** has a set of proxies—**Envoys**—deployed as **sidecars**. They _mediate and control network communication between microservices_ along with **istiod** (previously, Galley and Mixer). You can deploy Envoy proxies to each Kubernetes pod to work alongside the applications. In return, you don't need to load additional libraries or make changes to your applications, as they manage client-side load balancing, circuit breakers, logging, mTLS, etc.

- The **Control Plane** takes care of _configuring and managing of the Envoy proxies to route traffic_. 

Originally, the Control Plane was made up of three major elements: 

- Istio **[Pilot](https://istio.io/pt-br/docs/reference/commands/pilot-agent/)** communicates with the proxies and deploys routing and configuration rules.
- Istio **[Galley](https://istio.io/pt-br/docs/reference/commands/galley/)** takes care of telemetry, such as logs. (Note: this part used to be called **Mixer**).
- Istio **[Citadel](https://istio.io/pt-br/docs/tasks/security/citadel-config/)** is the security center that takes care of certificates for the proxies in the entire Istio network mesh.

Starting from v1.5 as of Q1 2020, the Istio components in the Control Plane were [consolidated](https://istio.io/docs/ops/deployment/architecture/) into **`istiod`**.

**`istiod`** unified Pilot, Galley, Citadel and the Envoy sidecar injector functionalities into a single binary. Installation became easier with fewer required Kubernetes deployments and associated configurations. Many of the configuration options are no longer needed. Things, such as VM usage, maintenance of dependencies, and scalability (with just one component versus several) became easier too.

> **Note**: In addition to being part of Anthos Service Mesh, Istio is also available as **[Istio on GKE](https://cloud.google.com/istio/docs/istio-on-gke/overview)** (a tool that provides automated installation and upgrade of Istio by running in GKE clusters).
> 
> Another key component of Anthos is **[Anthos Config Management](https://cloud.google.com/anthos-config-management/docs)** designed for automation of security and policy at scale. It allows to create a common configuration, including custom policies, and sync it across all Kubernetes clusters. With Anthos Config Management you get _policy as code_ thanks to the use of Git, YAML, and JSON.
> 
> In addition, Anthos uses the **[Operations Suite](https://cloud.google.com/stackdriver/docs)** (formerly, **Stackdriver**), which provides an overview of logs and telemetries sent by Istio and allows you see logs across all services. It also allows application debugging and incident management.

---

(An extract from the tutorial)

## A brief history of containers    

There is a distinct relationship between how the cloud services came to be—since the VMs started being rolled out in mid-2000s as a service—and the idea of application packaging and resource isolation.

Containers have been helping users separate applications from infrastructure and shorten the cycle between writing and running code. They have allowed monolithic applications inside virtual machines (VMs) to be split into smaller parts and run simultaneously, saving time and possibly a lot of money. With containerization, developers have been able to deploy and update software on the go.

A series of projects—from Google's Borg, CGroups, Docker, Kubernetes, and Knative to Istio—have introduced a number of key features that define modern on-prem and cloud services, namely:

- Immutability of infrastructure.
- Declarative configuration.
- Server-side deployment
- Health maintenance.
- Goal-oriented processes.

### _Borg_

Google has been running its software in containers for many years. In 2003, it began working on automating cluster management with a project titled **[Borg](https://research.google/pubs/pub43438/)**. It was a large scale cluster manager born out of the merger of two systems called **Global Work Queue** (for running batch jobs across many machines, such as MapReduce) and **Babysitter** (for restarting crashed servers).

### _CGroups/Process Containers_

In 2006-2007, engineers at Google started working on an open-source feature for the Linux kernel called **[CGroups](https://en.wikipedia.org/wiki/Cgroups)** (Control Groups, previously known as Process Containers) for resource or group process management. CGroups was originally mainlined into the Linux kernel in 2007.

### _Docker_

<center><img src="img/docker-moby-logo.png" width="100", hspace="20" vspace="20"></center>

The CGroups method of isolation and management of resources has been used in several ways, including through **[Docker](https://docs.docker.com/get-started/)**. Open-sourced in 2013, Docker is a platform for developing, shipping, and running applications. It introduced developer-friendly image distribution infrastructure and container runtime, and tools for image building and formatting.

Docker made it more flexible to manage and run containers by packaging applications and their dependencies into lightweight containers that can be run on-prem, in the cloud, or on VMs.

### _Kubernetes_

<center><img src="img/kubernetes-logo.png" width="110", hspace="20" vspace="20"></center>

Running many containers can be challenging. So, in 2014-2015, Google announced and open-sourced a project called **[Kubernetes (K8s)](https://kubernetes.io/)** for multiple container orchestration, which builds upon the company's experience running production workloads.

Kubernetes allows users to deploy software to many machines in orchestration and run containers at scale. With Kubernetes, applications inside containers can be run anywhere with the tools and resources they need.

Since its launch, Kubernetes has become the de facto standard for running containers.

### Istio 

<center><img src="img/istio-logo.png" width="120", hspace="20" vspace="20"></center>

As application architectures became increasingly based on shared services that are deployed and scaled dynamically on the cloud, there was a need for stronger service identity and authentication. As services are broken into atomic parts, it becomes more challenging to manage them. 

Therefore, in 2017, teams from Google, IBM and Lyft founded a project called **[Istio](https://istio.io/)**. It is an open-source platform for connecting, securing, controlling, and monitoring microservices, while reducing the complexity of managing their deployments. Istio allows developers to deploy site reliability engineering best practices. Version 1.0 of Istio was released in 2018. 

### _Knative_

<center><img src="img/knative-logo.png" width="110", hspace="20" vspace="20"></center>

Kubernetes is a powerful project. And applications that can be run outside of monolithic VMs in a serverless way have many benefits, including on the billing side. So, in 2018, Google brought Kubernetes and serverless together by open-sourcing a project called **[Knative](https://knative.dev/)**.

Knative extends Kubernetes and provides essential components for building source-centric, modern, and container-based software that can run anywhere, while bringing workload portability and the serverless developer experience. Users are able to have both the velocity of application development and flexibility without the lock-in. Knative has received [contributions](https://knative.dev/community/contributing//annual_reports/Knative%202019%20Annual%20Report.pdf) from Pivotal, IBM, and Red Hat, among others, and has had collaborations with the open-source function-as-a-service framework communities, such as OpenWhisk, riff, and Kyma.

> Knative powers Google Cloud's managed cloud-based solution called **[Cloud Run](https://cloud.google.com/run/docs)**. It is also a part of **[Cloud Run for Anthos](https://cloud.google.com/run/docs/gke/setup)**, which is not covered in this tutorial.
