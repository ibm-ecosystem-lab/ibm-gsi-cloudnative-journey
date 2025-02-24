---
title: Artifact Management
---
<!--- cSpell:ignore ICPA openshiftconsole Theia userid toolset crwexposeservice gradlew bluemix ocinstall Mico crwopenlink crwopenapp swaggerui gitpat gituser  buildconfig yourproject wireframe devenvsetup viewapp crwopenlink  atemplatized rtifactoryurlsetup Kata Koda configmap Katacoda checksetup cndp katacoda checksetup Linespace igccli regcred REPLACEME Tavis pipelinerun openshiftcluster invokecloudshell cloudnative sampleapp bwoolf hotspots multicloud pipelinerun Sricharan taskrun Vadapalli Rossel REPLACEME cloudnativesampleapp artifactoryuntar untar Hotspot devtoolsservices Piyum Zonooz Farr Kamal Arora Laszewski  Roadmap roadmap Istio Packt buildpacks automatable ksonnet jsonnet targetport podsiks SIGTERM SIGKILL minikube apiserver multitenant kubelet multizone Burstable checksetup handson  stockbffnode codepatterns devenvsetup newwindow preconfigured cloudantcredentials apikey Indexyaml classname  errorcondition tektonpipeline gradlew gitsecret viewapp cloudantgitpodscreen crwopenlink cdply crwopenapp -->

In IBM Garage Method, one of the Develop practices is to [automate continuous delivery through a delivery pipeline](https://www.ibm.com/garage/method/practices/deliver/practice_delivery_pipeline/), in part by using an artifact repository for storing output of the build stage. The <Globals name="env" /> uses Artifactory as an artifact repository manager, which it uses to host its Helm repository.

## What is Artifactory

[Artifactory](https://jfrog.com/artifactory/) is an artifact management repository.

An artifact repository manager hosts multiple binary repositories, like a database management system for artifacts. The binary repository compliments the source code repository: the code from an SCM is the input to the build process, whereas a binary repository stores the output of the build process, often called artifacts. The artifacts are often individual application components that can later be assembled into a full product.

An artifact repository manager is an integral part of a CI/CD solution, a companion to the pipeline. As the pipeline builds artifacts, they're stored in the repositories. When the pipeline later needs artifacts that have already been built, they're retrieved from the repositories. This enables a build to be broken into smaller stages with well-defined inputs and outputs and provides better tracking of each stage's results. Often a failed pipeline can restart in the middle using artifacts that were already built and stored.

An artifact repository often serves as the storage for a package manager, which assembles an application from artifacts. Here are some common package managers and their repositories:
- [**Maven**](https://maven.apache.org): Builds Java artifacts (such as Jar, War, Ear, etc.) and projects stored in [Maven repositories](https://maven.apache.org/guides/introduction/introduction-to-repositories.html) such as [Maven Central](https://maven.apache.org/repository/index.html)
- [**npm**](https://docs.npmjs.com/about-npm/): Assembles programs from JavaScript packages stored in [npm-registries](https://docs.npmjs.com/misc/registry) such as the [public npm registry](https://docs.npmjs.com/about-the-public-npm-registry)
- [**PIP**](https://pypi.org/project/pip/): Installs Python packages from [index repositories](https://packaging.python.org/guides/hosting-your-own-index/) such as the [Python Package Index (PyPI)](https://pypi.org/)
- [**Helm**](https://helm.sh): Deploys applications to Kubernetes using charts stored in [Helm repositories](https://helm.sh/docs/chart_repository/) such as the [Helm Hub](https://helm.sh/blog/intro-helm-hub/) catalog of repositories

Docker is not a package manager, but its architecture includes an artifact repository:
- [**Docker**](https://www.docker.com/): Stores images in [Docker registries](https://docs.docker.com/registry/) such as [Docker Hub](https://docs.docker.com/docker-hub/)

Note that you do not need a very large team to start reaping benefits from an artifact repository manager. The initial investment is not very large and the benefits are felt immediately.

## Artifact management in the Pipeline

The <Globals name="env" /> will eventually be extended to store a number of artifact types in Artifactory. Thus far, the CI and CD pipelines exchange two types of artifacts: Docker images and Helm charts. The CI pipeline ([Jenkins](../continuous-integration/), [Tekton](../../guides/continuous-integration-tekton/), etc.) builds these artifacts and ArgoCD deploys them. To store and share the artifacts, the pipeline uses two repositories:
- **Docker images**: The [Developer Tools Image Registry](../image-registry/)
- **Helm charts**: A Helm repository in Artifactory

The [<Globals name="templates" />](../../resources/codepatterns-overview) have also been configured to store their Helm charts in Artifactory.

Artifactory is part of the <Globals name="env" />'s complete CI/CD solution:



### Artifactory dashboard

Use the Artifactory dashboard to browse the repositories and their artifacts.

Open the Artifactory web UI for your environment.

- Go to the OpenShift console. On the top, you will find an icon of a square made of 9 small squares. 
- Clicking on that icon will give you all the tools shortcut. Select Artifactory to navigate to the Artifactory dashboard.

Browse the Helm repository.
- In the sidebar on the left, select Artifactory > Artifacts.
- Expand the tree for the `generic-local` repository, which is the Helm repository
- Expand the branch for your environment's cluster, such as `showcase-dev-iks`

The artifacts in the cluster's branch follow the [Helm chart repository](https://helm.sh/docs/chart_repository/) structure:
- **index.yaml**: Helm's index of all of the charts in the repository
- **charts**: The `tgz` files named for the application they deploy
      ![Artifact Repository Browser](images/Artifactory01.png)

Browse the artifacts to see how a Helm repository is organized.
- Select the `index.yaml` file and View it to see its contents
      ![Helm Repository Index](images/ArtifactoryIndexyaml.png)
- Expand a chart's branch to see that the `tgz` file contains the chart directory structure
      ![Helm Chart Structure](images/artifactoryuntar.png)

Notice that each chart has its own URL in Artifactory, and index lists the URL for a chart.

## Conclusion

The <Globals name="env" /> includes an artifact repository manager called Artifactory, which it uses to host a Helm repository.
As the CI pipeline builds the Helm chart for an application, it stores the chart in the Artifactory repository. When the ArgoCD pipeline deploys an application, it retrieves the chart from the Artifactory repository.
