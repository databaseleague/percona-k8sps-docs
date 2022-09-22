# System Requirements

The Operator supports Percona Server for MySQL (PS) 5.7 and 8.0.

## Officially supported platforms

The following platforms were tested and are officially supported by the Operator
{{ release }}:

* [Google Kubernetes Engine (GKE)](https://cloud.google.com/kubernetes-engine) 1.21 - 1.23

* [Amazon Elastic Container Service for Kubernetes (EKS)](https://aws.amazon.com) 1.19 - 1.22

* [Minikube](https://minikube.sigs.k8s.io/docs/) 1.26 (based on Kubernetes 1.24)

Other Kubernetes platforms may also work but have not been tested.

## Resource Limits

A cluster running an officially supported platform contains at least three
Nodes, with the following resources:

* 2GB of RAM,

* 2 CPU threads per Node for Pods provisioning,

* at least 60GB of available storage for Persistent Volumes provisioning.