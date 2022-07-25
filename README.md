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

---

### Deploy the service using yaml.

We assume that you are in the folder that you have clone/fork the code.

1. Login to you cluster using the CLI
2. Create a new project.
    ```
    oc new-project manifest-demo
    ```
3. Apply the 3 yaml files
    ```
    oc apply -f manifest-demo/deployment.yaml
    ```
    ```
    oc apply -f manifest-demo/service.yaml
    ```
    ```
    oc apply -f manifest-demo/route.yaml
    ```

4. Retrieve the URL
    ```
    echo "URL: $(oc  get route simple-quarkus-service --template='http://{{.spec.host}}')"
    ```
5. Take the url you have just retrieve with you favorite browser.

---

### Deploy Using OpenShift Pipeline

We assume that you are in the folder that you have clone/fork the code. For instructions on how to install OpenShift Pipiline you can refer to my [OpenShift Pipeline Demo](https://github.com/froberge/ocp-pipeline-demo) in this [section](https://github.com/froberge/ocp-pipeline-demo/blob/main/docs/install-pipeline-operator.md)

1. Login to you cluster using the CLI
1. Make sure you are in the pipeline-demo folder
`.../pipeline-demo/`

1. Using `Kustomize` create the different resources needed to run the demo
    ```
    oc apply -k pipeline-demo
    ```
    Result:
    ```
    namespace/pipeline-demo created
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
1. Let's expose the trigger service
    ```
    oc expose svc el-github-webhook -n pipeline-demo
    ```
    ```
    oc get route el-github-webhook -n pipeline-demo
    ```
1. Retrive the trigger url.
    ```
    echo "$(oc  get route el-github-webhook -n pipeline-demo  --template='http://{{.spec.host}}')"
    ```
1.  Create the GitHub Webhook

    Open [GitHub](https://github.com/)  in the right repository, go to setting -> Webhook -> Add Webhook

    _Before you click Add webhook it should look similar to this. With the URL retrieve step 5._

    ![Webhook](/docs/images/github-webhook.png)

    You can now push a change to the repository, it should trigger the pipeline.
    
1. Open OpenShift console and validate the the pipeline is running.

---

### Deploy Using OpenShift GitOps

We assume that you are in the folder that you have clone/fork the code. For instructions on how to install OpenShift Gitops you can refer to my [OpenShift GitOps Demo](https://github.com/froberge/ocp-gitops-demo) in this [section](https://github.com/froberge/ocp-gitops-demo/blob/main/docs/install-gitops-operator.md)


In this section we will be demontrating how to use `OpenShift` gitops to manage our application and to deploy the application. To to this we need to deploy an other `ArgoCD instance` that will be use for developers to manage the applications

__NOTE__
*   The default `cluster` instance of Argo CD is meant for cluster admin tasks like creating namespace managing role bindings not for day to day application management.

* `The Developer Argo CD instance` will be deploy in it own namespaces and is intented for the developper to use to manage the application.

1. Login to you cluster using the CLI

1. Use kKustomize` create the different resources needed to run the demo 
    ```
    oc apply -k gitops-demo/setup/overlays/demo
    ```

    This will create all the elements required    
        * __simple-quarkus-gitops__ - The namespace where the argoCD instance for Developer will be install.
        * __simple-quarkus-pipeline__ - The namespaces where the pipeline resources will be install.
        * __simple-quarkus-dev__ - The `DEV` environment for the demo.
        * __simple-quarkus-prod__ - The `PROD` environment for the demo.

#### SETUP GitHub

We need to set up 2 different elements in GitHub
    1. A personel Token on you profile
    2. A Webhook on the code repository


##### Creation of a Personal token creation

From your [GitHub](github.com) account.

1. Under profile Select `setting`.
    ![GitHub setting](docs/images/github-setting.png)

2. Select `Developer settings`

    ![GitHub dev](docs/images/github-devsetting.png)

3. Select `Personal access tokens`

    ![GitHub personal token](docs/images/github-personal-token.png)

4. Click `Generate new token` and enter the information.

    ![GitHub personal token 2](docs/images/github-access-token.png)

    * `Note`: A name for token
    * `Expiration`:When the token should expire according to you security policy.
    *  `Select scopes`: _Repo_ need to be selected at a minumun
    * Click `Generate token`

    > :warning: Copy the generated token in a secure place, since once the window is close, you won't be able to retrieve it. 

5. Generate the require secret for OpenShift.
    * You need to edit the file `../gitops-demo/manifest/github-secret.yaml`
    * Replace the information with your information
        * `username:` [Add your username]
        * `password:` [ADD GITHUB TOKEN HERE]
    * Apply the file to OpenShift
        ```
        oc apply -f gitops-demo/manifest/github-secret.yaml
        ```
---
##### Create the GitHub Webhook
1. Retrive the trigger url.     
    ```
    echo "$(oc  get route el-github-webhook -n simple-quarkus-pipeline  --template='http://{{.spec.host}}')"
    ```

2. Open [GitHub](https://github.com/)  in the the code repository, go to setting -> Webhook -> Add Webhook

    ![Webhook](/docs/images/github-webhook.png)

You can now push a change to the repository, it should trigger the pipeline. 

---

#### Make different modification to the application to test the pipeline.

* Modify the source code will trigger the pipeline
* Modify the manifest it will trigger the ArgoCD sync


