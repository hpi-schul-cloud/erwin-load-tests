# ErWIn-IDM load test

This guide is for developers of the schul-cloud and outlines the steps to set up and execute the ErWIn-IDM load test in your Kubernetes environment using k6, Lens, and Helm.

## Content and structure

### Folder 'js'

Consists of the k6 test script that will be referenced in the values.yaml.

### Folder 'templates'

Content of the Helm Chart to deploy.

### File 'values.yaml'

Parameters for dynamic reusability of the load test.
Those parameters will be used in 'templates/cronjob.yaml'.

## Prerequisites

### Helm

Helm is the package manager for Kubernetes and must be installed. Use this [guide](https://helm.sh/docs/intro/install/) or the following winget command.

```pwsh
winget install -e --id Helm.Helm
```

### Repository

Clone the *'erwin-load-tests'* repository from GitHub and enter the superhero credentials in the *'getAccessToken'* function in the *'server.spec.js'* file.

>[!WARNING] 
> Never push credentials!

### Lens

- **Download Kubernetes Configuration Files:**

    1. Download *'erwin-loadtest-kubeconfigs'* from 1Password.
    2. Unzip the downloaded file.
    3. Copy the two YAML files from the unzipped folder into your *.kube* directory. By default, this directory can be found under your user's home directory.

- **Add Required Kubernetes Namespaces:**

    Namespaces still need to be created. You can find a guide for this [here](https://docs.dbildungscloud.de/x/6YDbE).

  - In the *'sc-dev-loadtestdriver'* cluster, create the *'erwin'* namespace. This namespace is where our load test runs.
  - In the *'sc-dev-loadtestitem'* cluster, create the *'loadtest-01'* namespace. This namespace is where the schul-cloud instance runs.

- **Configure Environment Variables and Ingress Paths:**

    It's important to manually configure environment variables and ingress paths. Note that you'll need to perform this process after every rollout of 'dof_app_deploy' because direct interventions in Lens are overwritten with each rollout.

  - To enable Keycloak for the load test, set the environment variables 'FEATURE_IDENTITY_MANAGEMENT_STORE_ENABLED' and 'FEATURE_IDENTITY_MANAGEMENT_LOGIN_ENABLED' to 'true' in the *'sc-dev-loadtestitem'* cluster. You can do this by editing the *'api-configmap'* under *'ConfigMaps'*.
  - To unlock the correct ingress paths, edit *'loadtest-01-ingress'* under *'Ingresses'* and add the following to *'paths'*.

      ```yaml
      - path: /api/v3/authentication
          pathType: Prefix
          backend:
          service:
              name: api-svc
              port:
                  number: 3030
      - path: /api/v3/account
          pathType: Prefix
          backend:
          service:
              name: api-svc
              port:
                  number: 3030
      ```

## Running the load test

1. Open Lens and navigate to the *'sc-dev-loadtestdriver'* cluster. Under the *'Helm'* section, locate the *'Releases'* tab. Delete the existing *'erwinloadtest'* release if it exists.
2. Open your terminal and navigate to the directory where the repository for the load test is located. Execute the following Helm installation command.

    ```pwsh
    helm install --kubeconfig /Users/<your user>/.kube/sc-dev-loadtestdriver.yaml -n erwin erwin-loadtest
    ```

    This command installs the 'erwin-loadtest' Helm chart on the 'sc-dev-loadtestdriver' cluster. Ensure you replace the path to your kubeconfig file.
3. Return to the *'sc-dev-loadtestdriver'* cluster in Lens. Go to the *'CronJobs'* section and trigger the *'loadtesterwin-server-spec'* CronJob. This action initiates the load test.
4. To monitor the progress of the load test, you can access the relevant information in the Pod Logs. These logs will provide insights into the performance of the load test.

    In addition, the Grafana dashboard *'Erwin Keycloak Metrics'* was created in all instances.

> [!NOTE]
> To simulate higher loads, you can increase the number of pods within the respective deployment. To do this, simply raise the *'replicas'* count under the *'spec'* section.
