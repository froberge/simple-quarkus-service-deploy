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


### Deploy the service using yaml.

We assume that you are in the folder that you have clone/fork the code.

1. Login to you cluster using the CLI
2. Create a new project.
    ```
    oc new-project manifest-demo
    ```
3. Apply the 3 yaml files
    ```
    oc apply -f manifest/simple-quarkus-service/deployment.yaml
    ```
    ```
    oc apply -f manifest/simple-quarkus-service/service.yaml
    ```
    ```
    oc apply -f manifest/simple-quarkus-service/route.
4. Retrieve the URL
    ```
    echo "URL: $(oc  get route simple-quarkus-service --template='http://{{.spec.host}}')"
    ```
5. Take the url you have just retrieve with you favorite browser.


### Deploy Using OpenShift Pipeline

We assume that you are in the folder that you have clone/fork the code.

1. Login to you cluster using the CLI
2. Create a new project.
    ```
    oc apply -f manifest/tekton/tekton-demo-namespace.yaml
    ```
3. Using `Kustomize` create the different resources needed to run the demo
    ```
     oc apply -k manifest/tekton
    ```
    result:
    ```
    service/simple-quarkus-service created
    deployment.apps/simple-quarkus-service created
    route.route.openshift.io/simple-quarkus-service created
    pipeline.tekton.dev/simple-quarkus-service created
    eventlistener.triggers.tekton.dev/github-webhook created
    triggerbinding.triggers.tekton.dev/github-pr-trigger-binding created
    triggertemplate.triggers.tekton.dev/simple-quarkus-service-trigger-template created
    persistentvolumeclaim/maven-repo-pvc created
    persistentvolumeclaim/sources-pvc created
    ```
4. Let's expose the trigger service
    ```
    oc expose svc el-github-webhook
    ```
    ```
    oc get route el-github-webhook
    ```
5. Retrive the trigger url.
```
echo "URL: $(oc  get route el-github-webhook --template='http://{{.spec.host}}')"
```
6. Create the GitHub Webhook

    Open [GitHub](https://github.com/)  in the right repository, go to setting -> Webhook -> Add Webhook

    Before you click Add webhook it should look similar to this. With the URL retrieve step 5.

    ![Webhook](/docs/images/github-webhook.png)

    You can now push a change to the repository, it should trigger the pipeline.
7. Open OpenShidt console and validate the the pipeline is running.

### Deploy Using OpenShift GitOps

We assume that you are in the folder that you have clone/fork the code.

1. Login to you cluster using the CLI
2. Create the argocd project 
    ```
    oc apply -f manifest/argocd-app/argocd-simple-demo.yaml
    ```
3. Connect to OpenShift Web Console you should now have access to the project and the service
4. Connect to argoCD you should see the argoCD project.
5. Commit change to the deployment file and see what happens.
6. Do the same with the UI.

:warning: By default the argoCD is not on auto healing, this is a change you should make to show the behavior.
