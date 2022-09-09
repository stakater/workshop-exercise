### Helm 101

> Helm is the package manager for Kubernetes. It provides a way to create templates for the Kubernetes YAML that defines our application. The Kubernetes resources such as `DeploymentConfig`, `Route` & `Service` can be processed by supplying `values` to the templates. In Helm land, there are a few ways to do this. A package containing the templates and their default values is called a `chart`. 

Let's deploy a simple application using Helm.

1. Helm charts are packaged and stored in repositories. They can be added as dependencies of other charts or used directly. Let's add a chart repository now. The chart repository stores the version history of our charts as well as the packaged tar file.

    ```bash#test
    helm repo add tl500 https://rht-labs.com/todolist/
    ```

2. Let's install a chart from this repo. First search the repository to see what is available.

    ```bash#test
    helm search repo todolist
    ```

    Now install the latest version. Helm likes to give each install a release, for convenience we've set ours to `my`. This will add a prefix of `my-` to all the resources that are created.

    ```bash#test
    helm install my tl500/todolist --namespace ${TEAM_NAME}-ci-cd || true
    ```

3. Open the application up in the browser to verify it's up and running. Here's a handy one-liner to get the address of the app

    ```bash#test
    echo https://$(oc get route/my-todolist -n ${TEAM_NAME}-ci-cd --template='{{.spec.host}}')
    ```

    ![todolist](./images/todolist.png)

4. You can overwrite the default <span style="color:blue;">[values](https://github.com/rht-labs/todolist/blob/master/chart/values.yaml)</span> in a chart from the command line. Let's upgrade our deployment to show this. We'll make a simple change to the values to scale up our app. By default, we only have 1 replica.

    ```bash#test
    oc get pods -n ${TEAM_NAME}-ci-cd
    ```

    By default, we only have one replica of our application. Let's use helm to set this to 5.

    ```bash#test
    helm upgrade my tl500/todolist --set replicas=5 --namespace ${TEAM_NAME}-ci-cd
    ```

    Verify the deployment has scaled up to 5 replicas.

    ```bash#test
    oc get pods -n ${TEAM_NAME}-ci-cd
    ```

5. If you're done playing with the #amazing-todolist-app then let's tidy up our work by removing the chart. To do this, run helm uninstall to remove our release of the chart.

    ```bash#test
    helm uninstall my --namespace ${TEAM_NAME}-ci-cd
    ```

    Verify the clean up

    ```bash#test
    oc get pods -n ${TEAM_NAME}-ci-cd | grep todolist
    ```

6. For those who are really interested, this is the anatomy of our Helm chart. It can be <span style="color:blue;">[found here](https://github.com/rht-labs/todolist)</span>, but the basic structure is as follows:

    <div class="highlight" style="background: #f7f7f7">
    <pre><code class="language-bash">
    todolist/chart
    ├── Chart.yaml
    ├── templates
    │   ├── _helpers.tpl
    │   ├── deploymentconfig.yaml
    │   ├── route.yaml
    │   └── service.yaml
    └── values.yaml
    </code></pre></div>

    where:
    * `Chart.yaml` - is the manifest of the chart. It defines the name, version and dependencies for our chart.
    * `values.yaml` - is the sensible defaults for our chart to work, it contains the variables that are passed to the templates. We can overwrite these values on the command line.
    * `templates/*.yaml` - they are our k8s resources. 
    * `_helpers.tpl` - is a collection of reusable variables an yaml snippets that are applied across all of the k8s resources uniformly for example, labels are defined in here and included on each k8s resource file as necessary.

🪄🪄 Now, let's continue with even more exciting tools... !🪄🪄