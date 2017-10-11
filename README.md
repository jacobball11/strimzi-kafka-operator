# Kafka as a Service

This project provides a way to run an Apache Kafka cluster on Kubernetes and OpenShift with different types of deployment.

<!-- TOC -->

- [Kafka as a Service](#kafka-as-a-service)
    - [Kafka Stateful Sets](#kafka-stateful-sets)
        - [Deploying to OpenShift](#deploying-to-openshift)
    - [Kafka in-memory](#kafka-in-memory)
        - [Deploying to OpenShift](#deploying-to-openshift-1)
    - [Kafka Connect](#kafka-connect)
        - [Deploying to OpenShift](#deploying-to-openshift-2)
        - [Deploying to Kubernetes](#deploying-to-kubernetes)

<!-- /TOC -->

## Kafka Stateful Sets

This deployment uses the Stateful Sets (previously known as "Pet Sets") feature of Kubernetes / OpenShift. With Stateful Sets, the pods receive unique name and network identity and that makes it easier to identify the individual Kafka broker pods and set their identity (broker ID). Each Kafka broker pod is using its own Persistent Volume. The Persistent Volume is acquired using Persistent Volume Claim - that makes it independent on the actual type of the Persistent Volume. For example it can use HostPath volumes on Minikube or Amazon EBS volumes in Amazon AWS deployments without any changes in the YAML files.

It's important to say that in this deployment both "regular" and "headless" services are used. The "regular" services are needed for having instances accessible from clients;
the "headless" services are needed for having the DNS resolving directly the PODs IP addresses for having the Stateful Sets working properly.

This deployment is available under the _kafka-statefulsets_ folder and provides following artifacts :

* Dockerfile : Docker file for building an image with Kafka and Zookeeper already installed
* config : configuration file templates for running Zookeeper
* scripts : scripts for starting up Kafka and Zookeeper servers
* resources : provides all YAML configuration files for setting up volumes, services and deployments

### Deploying to OpenShift

To conveniently deploy a StatefulSet to OpenShift a [Template is provided](kafka-statefulsets/resources/openshift-template.yaml), this could be used via `oc create -f kafka-statefulsets/resources/openshift-template.yaml` so it is present within your project. To create a whole Kafka StatefulSet use `oc new-app barnabas`.

## Kafka in-memory

Kafka in-memory deployment is just for development and testing purpose and not for production. It is designed the same way as the Kafka Stateful Sets deployment. The only difference is that for storing broker information (Zookeeper side) and topics/partitions (Kafka side), an _emptyDir_ is used instead of Persistent Volume Claims. This means that its content is strictly related to the pod life cycle (deleted when the pod goes down). However for development and testing it might be easier to use the in-memory deployment because it doesn't need to provision the persistent volumes needed for the stateful set deployment.

This deployment is available under the _kafka-inmemory_ folder and provides following artifacts :

* resources : provides all YAML configuration files for setting up services and deployments

### Deploying to OpenShift

To conveniently deploy a StatefulSet to OpenShift a [Template is provided](kafka-inmemory/resources/openshift-template.yaml), this could be used via `oc create -f kafka-inmemory/resources/openshift-template.yaml` so it is present within your project. To create a whole Kafka in-memory cluster use `oc new-app barnabas`.

## Kafka Connect

This deployment adds Kafka Connect cluster which can be used with the Kafka deployments above. It is implemented as deployment with a configurable number of workers. The default image currently contains only the Connectors which are part of Apache Kafka project - `FileStreamSinkConnector` and `FileStreamSourceConnector`. The REST interface for managing the Kafka Connect cluster is exposed internally within the Kubernetes / OpenShift cluster as service `kafka-connect` on port `8083`.

If you want to use other connectors, you can:
* Mount a volume containing the plugins to path `/opt/kafka/plugins/`
* Use the `enmasseproject/kafka-connect` image as Docker base image, add your connectors to the `/opt/kafka/plugins/` directory and use this new image instead of `enmasseproject/kafka-connect`

### Deploying to OpenShift

To conveniently deploy Kafka Connect to OpenShift a [Template is provided](kafka-connect/resources/openshift-template.yaml), this could be used via `oc create -f kafka-connect/resources/openshift-template.yaml` so it is present within your project. To create a whole Kafka Connect cluster use `oc new-app barnabas-connect` *(make sure you have already deployed a Kafka broker)*.

### Deploying to Kubernetes

To conveniently deploy Kafka Connect to Kubernetes use the provided yaml files with the Kafka Connect [deployment](kafka-connect/resources/kafka-connect.yaml) and [service](kafka-connect/resources/kafka-connect-service.yaml). This could be done using `kubectl apply -f kafka-connect/resources/kafka-connect.yaml` and `kubectl apply -f kafka-connect/resources/kafka-connect-service.yaml`.
