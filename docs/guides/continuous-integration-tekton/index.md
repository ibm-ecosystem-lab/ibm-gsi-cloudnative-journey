---
title: Continuous Integration with Tekton
description: Use Tekton to automate your continuous integration process
---

<!--- cSpell:ignore Opensource Tetkon ICPA openshiftconsole Theia userid toolset crwexposeservice gradlew bluemix ocinstall Mico crwopenlink crwopenapp swaggerui gitpat gituser  buildconfig yourproject wireframe devenvsetup viewapp crwopenlink  atemplatized rtifactoryurlsetup Kata Koda configmap Katacoda checksetup cndp katacoda checksetup Linespace igccli regcred REPLACEME Tavis pipelinerun openshiftcluster invokecloudshell cloudnative sampleapp bwoolf hotspots multicloud pipelinerun Sricharan taskrun Vadapalli Rossel REPLACEME cloudnativesampleapp artifactoryuntar untar Hotspot devtoolsservices Piyum Zonooz Farr Kamal Arora Laszewski  Roadmap roadmap Istio Packt buildpacks automatable ksonnet jsonnet targetport podsiks SIGTERM SIGKILL minikube apiserver multitenant kubelet multizone Burstable checksetup handson  stockbffnode codepatterns devenvsetup newwindow preconfigured cloudantcredentials apikey Indexyaml classname  errorcondition tektonpipeline gradlew gitsecret viewapp cloudantgitpodscreen crwopenlink cdply crwopenapp -->

## Overview

Continuous integration is a software development technique where software is built regularly by a team in an automated fashion.

Tekton is a new emerging CI tool that has been built to support Kubernetes and OpenShift from the ground up.

## What is Tekton

[Tekton](https://tekton.dev/) is a powerful yet flexible Kubernetes-native open-source framework for creating continuous integration and delivery (CI/CD) systems. It lets you build, test, and deploy across multiple cloud providers or on-premises systems by abstracting away the underlying implementation details.

### Tekton 101

<iframe width="80%" height="315" src="https://www.youtube.com/embed/TWxKD9dLpmk" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

Tekton provides open-source components to help you standardize your CI/CD tooling and processes across vendors, languages, and deployment environments. Industry specifications around pipelines, releases, workflows, and other CI/CD components available with Tekton will work well with existing CI/CD tools such as Jenkins, Jenkins X, Skaffold, and Knative, among others.

- For more information read up about it [Tekton Tutorial](https://developer.ibm.com/tutorials/knative-build-app-development-with-tekton/)

- For more information read up about it [App Build Tutorial with Tekton](https://developer.ibm.com/tutorials/knative-build-app-development-with-tekton/)

The IBM Cloud is standardizing on using Tekton in both IBM Cloud DevOps
 service and IBM Cloud Pak for Applications. OpenShift 4.2 will also natively
  support it.

This guide will focus on using Tekton when the Development tools have been
 installed in Redhat OpenShift, IBM Kubernetes Managed services and **Red Hat
  Code Ready Containers** to give you choice for you Continuous Integration
   Cloud-Native development toolset.

!!! note

    **Note:** This guide will help you set up the [<Globals name="templates" />](/resources/codepatterns-overview) with  **Tekton** and requires that you have installed Tekton with **Red Hat Code Ready Containers 4.2** or have installed open source Tekton into the The IBM Kubernetes Cluster.



### Common App Tasks

The following gives a description of each `Task` that is commonly used in a
 `Pipeline`. The *Optional* stages can be deleted or ignored if the tool support it is not installed. These stages represent a typical production pipeline flow for a Cloud-Native application.

- **Setup** clones the code into the pipeline
- **Build** runs the build commands for the code
- **Test**	validates the unit tests for the code
- **Publish pacts**	(*optional*) publishes any pact contracts that have been defined
- **Verify pact** (*optional*) verifies the pact contracts
- **Sonar scan** (*optional*) runs a sonar code scan of the source code and publishes the results to SonarQube
- **Build image** Builds the code into a Docker images and stores it in the IBM Cloud Image registry
- **Deploy to DEV env**	Deploys the Docker image tagged version to `dev` namespace using Helm Chart
- **Health Check** Validates the Health Endpoint of the deployed application
- **Package Helm Chart** (*optional*) Stores the tagged version of the Helm chart into Artifactory
- **Trigger CD Pipeline** (*optional*) This is a GitOps stage that will
 update the build number in designated git repo and trigger ArgoCD for
  deployment to **test** namespace

### Install Tekton

Tekton can be installed in both RedHat Openshift and IBM Kubernetes manage
 service and RedHat Code Ready Containers. To install the necessary
  components follow the steps below.

- Install IBM Garage for Cloud Developer Tools on your
 managed OpenShift,CRC or IKS development cluster on the IBM Cloud. This will
  help configure the tools and `secrets` and `configMap` to make working with
   IBM Cloud so much easier.


=== "OpenShift 4.x"

    ### Install on OpenShift 4.x

    - If you have installed the IBM Garage for Cloud Developer Tools into your cluster this will automatically install the operator for you.

    - Install Tekton on OpenShift 4 including CodeReady Containers (CRC)
    - Install via operator hub Administrator perspective view, click
      **Operator Hub** search for `OpenShift Pipelines` and install operator


=== "Kubernetes"

    ### Install Tekton on IBM Kubernetes Managed Service
    - Install Tekton via Knative add-on (can be found in the **Add On** tab in the Kubernetes managed service dashboard)
    , it includes
    Tekton support by default and the Webhook extension.
    - Install [Tekton Dashboard](https://github.com/tektoncd/dashboard#install-dashboard) follow the instructions in the `README.md`
    - Add Ingress endpoint for the **Tekton Dashboard** add a host name that uses the IKS cluster subdomain
    ```yaml
      apiVersion: extensions/v1beta1
      kind: Ingress
      metadata:
        name: tekton-dashboard
        namespace: tekton-pipelines
      spec:
        rules:
        - host: tekton-dashboard.showcase-dev-iks-cluster.us-south.containers.appdomain.cloud
          http:
            paths:
            - backend:
                serviceName: tekton-dashboard
                servicePort: 9097
     ```
     - Install [Tekton Webhook Extension](https://github.com/tektoncd/experimental/tree/master/webhooks-extension#install-webhook-extension)

### Setup Tekton

- Install Tekton pipelines and tasks into the `dev` namespace following the
 instructions in the repository [ibm-garage-tekton-tasks](https://github.com/ibm-garage-cloud/ibm-garage-tekton-tasks/blob/master/README.md)
- Install the `Tasks`
    ```bash
    kubectl create -f ibm-garage-tekton-tasks/tasks/ -n dev
    ```
- Install the `Pipelines`
    ```bash
    kubectl create -f ibm-garage-tekton-tasks/pipelines/ -n dev
    ```

### Configure namespace for development

- Install the Tekton CLI `tkn` https://github.com/tektoncd/cli

- Create a new namespace (ie `dev-{initials}`) and copy all config and secrets
  ```
  igc namespace -n {new-namespace}
  ```
- Set your `new-namespace` the current namespace context
  ```
  oc project {new-namespace}
  ```

- The template `Pipelines` provided support for `Java` or `Node.js` based apps. You can configure your own custom `Tasks` for other runtimes. There are a number of default `Tasks` to get you started they are detailed above. To create an application use one of the provided [<Globals name="templates" />](/resources/codepatterns-overview) these templates work seamlessly with the `Tasks` and `Pipelines` provided.

### Register the App with Tekton

With Tetkon enabled and your default `Tasks` and `Pipelines` installed into
 the `dev` namespace. You can now configure your applications to be built, packaged, tested and deployed to your OpenShift or Kubernetes development cluster.

- Connect to the pipeline. (See the [IGC CLI](/getting-started/cli) for details about how the `pipeline` command works.)

    ```bash
    igc pipeline -n dev-{initials} 
    ```

### Verify the pipeline

To validate your pipeline have been correctly configured, and has triggered a
 `PipelineRun`
 use the following **Tekton** dashboards or `tkn` CLI to validate it ran
  correctly without errors.


=== "OpenShift 4.x"

    - Review you **Pipeline** in the OpenShift 4.x Console
    ![Pipelinerun](../../images/deploy-app/pipeline.png)

    - Review your **Tasks**
    ![Tasks](images/tasks.png)

    - Review your **Steps**
    ![Steps](images/steps.png)

=== "Opensource Tekton Dashboard"

    If you are running Tekton with IBM Cloud Pak for Applications or Knative with Kubernetes managed service your dashboard view will look similar to below.

    - Review your **Pipeline**
    ![PipelineRun](images/pipeline-os.png)


=== "Tekton CLI"

    If you are running **Tekton** with IBM Cloud Pak for Applications or Knative with Kubernetes managed service your dashboard view will look similar to below.

    - Review your **Pipeline**
    ```bash
    tkn pipelinerun list
    ```
    - Review **Pipeline** details
    ```bash
    tkn pipelinerun describe {pipeline-name}
    ```

### Running Application

Once the **Tekton** pipeline has successfully completed you can validate your
 app has been successfully deployed.

- Open the OpenShift Console and select the {new-namespace} project and click on **Workloads**
    ![OpenShift](images/openshiftconsole.png)

- Get the hostname for the application from ingress
    ```bash
    kubectl get ingress --all-namespace
    ```
- You can use the the `igc` command to get the name of the deployed application
    ```bash
    igc ingress -n {new-namespace}
    ```

- Use the application URL to open it your browser for testing

Once you become familiar with deploying code into OpenShift using **Tekton
**, read up about how you can manage code deployment with `Continuous
 Delivery` with `ArgoCD` and `Artifactory`




