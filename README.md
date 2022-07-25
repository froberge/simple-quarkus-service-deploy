# Simple Quarkus Service Deploy

This repository contains the different manifest needed to deploy the [Simple Quarkus Service](https://github.com/froberge/simple-quarkus-service) code to an OpenShift/Kubernetes clusters.


This repository is used to implement the `best pratices` to seperate the code from the deployment manifest.

It is use for demo purposes and can deploy the Simple Quarkus Service in 3 differents ways.

### Prerequisite

1. Access to github
1. Clone/Fork of this repository
1. Access to an OpenShift Cluster
1. OpenShift CLI
1. [Optional] OpenShift Pipeline operator install on the cluster to run the Tekton demo
1. [Optional] OpenShift GitOps operator install on the cluster to run the GitOps demo


### Content

* [Deploy the service using simple yaml](docs/deploy-simple-manifest.md).
* [Deploy using OpenShift pipeline](docs/deploy-ocp-pipeline.md)
* [Deploy using OpenShift GitOps](docs/deploy-ocp-gitops.md)
