# Upgrade Database and Operator

Starting from the version 0.6.0, Percona Operator for MySQL based on Percona
Server for MySQL fully supports upgrades to newer versions. The upgradable
components of the cluster are the following ones:

* the Operator;
* [Custom Resource Definition (CRD)](operator.md),
* Database Management System (Percona Server for MySQL).

The list of recommended upgrade scenarios includes two variants:

* Upgrade to the new versions of the Operator *and* Percona Server for MySQL,
* Minor Percona Server for MySQL version upgrade *without* the Operator upgrade.

## Upgrading the Operator and CRD

!!! note

    The Operator supports **last 3 versions of the CRD** including the newest
    one, so it is technically possible to skip upgrading the CRD and just
    upgrade the Operator. If the CRD version is one of these, you will be able
    to continue using the old CRD and even carry on Percona Server for MySQL
    minor version upgrades with it. But the recommended way is to update the
    Operator *and* CRD.

Only the incremental update to a nearest version of the Operator is supported
(for example, update from 0.5.0 to 0.6.0). To update to a newer version, which
differs from the current version by more than one, make several incremental
updates sequentially.

### Manual upgrade

The upgrade includes the following steps.

1. Update the [Custom Resource Definition](https://kubernetes.io/docs/concepts/extend-kubernetes/api-extension/custom-resources/)
    for the Operator, taking it from the official repository on Github, and do
    the same for the Role-based access control:

    ``` {.bash data-prompt="$" }
    $ kubectl apply -f https://raw.githubusercontent.com/percona/percona-server-mysql-operator/v{{ release }}/deploy/crd.yaml
    $ kubectl apply -f https://raw.githubusercontent.com/percona/percona-server-mysql-operator/v{{ release }}/deploy/rbac.yaml
    ```

2. Now you should [apply a patch](https://kubernetes.io/docs/tasks/run-application/update-api-object-kubectl-patch/) to your
    deployment, supplying necessary image name with a newer version tag. You can find the proper
    image name for the current Operator release [in the list of certified images](images.md)
    (for older releases, please refer to the [old releases documentation archive](https://docs.percona.com/legacy-documentation)).
    For example, updating to the `{{ release }}` version should look as
    follows.

    ``` {.bash data-prompt="$" }
    $ kubectl patch deployment percona-server-mysql-operator \
      -p'{"spec":{"template":{"spec":{"containers":[{"name":"percona-server-mysql-operator","image":"percona/percona-server-mysql-operator:{{ release }}"}]}}}}'
    ```

3. The deployment rollout will be automatically triggered by the applied patch.
    You can track the rollout process in real time with the
    `kubectl rollout status` command with the name of your cluster:

    ``` {.bash data-prompt="$" }
    $ kubectl rollout status deployments percona-server-mysql-operator
    ```

    !!! note

        Labels set on the Operator Pod will not be updated during upgrade.

### Upgrade via helm

If you have [installed the Operator using Helm](helm.md), you can upgrade the
Operator with the `helm upgrade` command.

1. In case if you installed the Operator with no [customized parameters](https://github.com/percona/percona-helm-charts/tree/main/charts/ps-operator#installing-the-chart), the upgrade can be done as follows: 

    ``` {.bash data-prompt="$" }
    $ helm upgrade my-op percona/ps-operator --version {{ release }}
    ```

    The `my-op` parameter in the above example is the name of a [release object](https://helm.sh/docs/intro/using_helm/#three-big-concepts)
    which which you have chosen for the Operator when installing its Helm chart.

    If the Operator was installed with some [customized parameters](https://github.com/percona/percona-helm-charts/tree/main/charts/ps-operator#installing-the-chart), you should list these options in the upgrade command.
    
    
    !!! note
    
        You can get list of used options in YAML format with the `helm get values my-op -a > my-values.yaml` command, and this file can be directly passed to the upgrade command as follows:

        ``` {.bash data-prompt="$" }
        $ helm upgrade my-op percona/ps-operator --version {{ release }} -f my-values.yaml
        ```

2. Update the [Custom Resource Definition](https://kubernetes.io/docs/concepts/extend-kubernetes/api-extension/custom-resources/)
    for the Operator, taking it from the official repository on Github, and do
    the same for the Role-based access control:

    ``` {.bash data-prompt="$" }
    $ kubectl apply -f https://raw.githubusercontent.com/percona/percona-server-mysql-operator/v{{ release }}/deploy/crd.yaml
    $ kubectl apply -f https://raw.githubusercontent.com/percona/percona-server-mysql-operator/v{{ release }}/deploy/rbac.yaml
    ```

!!! note

    You can use `helm upgrade` to upgrade the Operator only. The Database Management System (Percona Server for MySQL) should be upgraded in the same way whether you used helm to install it or not.

## Upgrading Percona Server for MySQL

The following section presumes that you are upgrading your cluster within the
*Smart Update strategy*, when the Operator controls how the objects
are updated. Smart Update strategy is on when the `updateStrategy` key in the
[Custom Resource](operator.md) configuration file is set to `SmartUpdate`
(this is the default value and the recommended way for upgrades).

!!! note

    As an alternative, the `updateStrategy` key can be set to `RollingUpdate` 
    and `OnDelete`. You can find out more about it in the
    [appropriate section](update.md#more-on-upgrade-strategies).

### Manual upgrade

Manual update of Percona Server for MySQL can be done as follows:

1. Make sure that `spec.updateStrategy` option in the [Custom Resource](operator.md)
    is set to `SmartUpdate`, `spec.upgradeOptions.apply` option is set to `Never`
    or `Disabled` (this means that the Operator will not carry on upgrades
    automatically).
    
    ```yaml
    ...
    spec:
      updateStrategy: SmartUpdate
      upgradeOptions:
        apply: Disabled
        ...
    ```

2. Now [apply a patch](https://kubernetes.io/docs/tasks/run-application/update-api-object-kubectl-patch/)
    to your Custom Resource, setting necessary Custom Resource version and image
    names with a newer version tag.

    !!! note

        Check the version of the Operator you have in your Kubernetes
        environment. Please refer to the [Operator upgrade guide](update.md#upgrading-the-operator)
        to upgrade the Operator and CRD, if needed.

    Patching Custom Resource is done with the `kubectl patch ps` command.
    Actual image names can be found [in the list of certified images](images.md)
    (for older releases, please refer to the [old releases documentation archive](https://docs.percona.com/legacy-documentation)).
    For example, updating `cluster1` cluster to the `{{ release }}` version
    should look as follows:

    ```bash
    $ kubectl patch ps cluster1 --type=merge --patch '{
       "spec": {
           "crVersion":"{{ release }}",
           "mysql":{ "image": "percona/percona-server:{{ ps80recommended }}" },
           "proxy":{
              "haproxy":{ "image": "percona/haproxy:{{ haproxyrecommended }}" },
              "router":{ "image": "percona/percona-mysql-router:{{ routerrecommended }}" }
           },
           "orchestrator":{ "image": "percona/percona-orchestrator:{{ orchestratorrecommended }}" },
           "backup":{ "image": "percona/percona-xtrabackup:{{ pxbrecommended }}" },
           "toolkit":{ "image": "percona/percona-server-mysql-operator:{{ release }}-toolkit" },
           "pmm": { "image": "percona/pmm-client:{{ pmm2recommended }}" }
       }}'
    ```

    !!! warning

        The above command upgrades various components of the cluster including PMM Client. It is [highly recommended](https://docs.percona.com/percona-monitoring-and-management/how-to/upgrade.html) to upgrade PMM Server **before** upgrading PMM Client. If it wasn't done and you would like to avoid PMM Client upgrade, remove it from the list of images, reducing the last of two patch commands as follows:        
        
        ```bash
        $ kubectl patch ps cluster1 --type=merge --patch '{
           "spec": {
               "crVersion":"{{ release }}",
               "mysql":{ "image": "percona/percona-server:{{ ps80recommended }}" },
               "proxy":{
                  "haproxy":{ "image": "percona/haproxy:{{ haproxyrecommended }}" },
                  "router":{ "image": "percona/percona-mysql-router:{{ routerrecommended }}" }
               },
               "orchestrator":{ "image": "percona/percona-orchestrator:{{ orchestratorrecommended }}" },
               "backup":{ "image": "percona/percona-xtrabackup:{{ pxbrecommended }}" },
               "toolkit":{ "image": "percona/percona-server-mysql-operator:{{ release }}-toolkit" }
           }}'
        ```

3. The deployment rollout will be automatically triggered by the applied patch.
    You can track the rollout process in real time with the
    `kubectl rollout status` command with the name of your cluster:

    ```default
    $ kubectl rollout status sts cluster1-ps
    ```

### Automated upgrade

*Smart Update strategy* allows you to automate upgrades even more. In this case
the Operator can either detect the availability of the new Percona Server for
MySQL version, or rely on the user's choice of the version. To check the
availability of the new version, the Operator will query a special
*Version Service* server at scheduled times to obtain fresh information about
version numbers and valid image paths.

If the current version should be upgraded, the Operator updates the Custom
Resource to reflect the new image paths and carries on sequential Pods deletion,
allowing StatefulSet to redeploy the cluster Pods with the new image.
You can configure Percona Server for MySQL upgrade via the `deploy/cr.yaml`
configuration file as follows:

1. Make sure that `spec.updateStrategy` option is set to `SmartUpdate`.

2. Change `spec.crVersion` option to match the version of the Custom Resource
    Definition upgrade [you have done](update.md#manual-upgrade) while upgrading
    the Operator:

    ```yaml
    ...
    spec:
      crVersion: {{ release }}
      ...
    ```
    
    !!! note

        If you don't update crVersion, minor version upgrade is the only one to
        occur. For example, the image `percona-server:8.0.30-22` can
        be upgraded to `percona-server:8.0.32-24`.

3. Change the `upgradeOptions.apply`  option from `Disabled` to one of the
    following values:

    * `Recommended` - [scheduled](operator.md#upgradeoptions-schedule) upgrades
        will choose the most recent version of software flagged as "Recommended"

    * `Latest` - automatic upgrades will choose the most recent version of
        the software available,

    * *version number* - specify the desired version explicitly
        (version numbers are specified as `{{ ps80recommended }}`, etc.).
        Actual versions can be found [in the list of certified images](images.md)
        (for older releases, please refer to the [old releases documentation archive](https://docs.percona.com/legacy-documentation)).

4. Make sure the `upgradeOptions.versionServiceEndpoint` key is set to a valid
    Version Server URL (otherwise upgrades will not occur).

    === "Percona’s Version Service (default)"
        You can use the URL of the official Percona’s Version Service (default).
        Set `upgradeOptions.versionServiceEndpoint` to `https://check.percona.com`.

    === "Version Service inside your cluster"
        Alternatively, you can run Version Service inside your cluster. This
        can be done with the `kubectl` command as follows:

        ``` {.bash data-prompt="$" }
        $ kubectl run version-service --image=perconalab/version-service --env="SERVE_HTTP=true" --port 11000 --expose
        ```

    !!! note

        Version Service is never checked if automatic updates are disabled in 
        the `upgradeOptions.apply` option. If automatic updates are enabled, but
        the Version Service URL can not be reached, no updgrades will be
        performed.

5. Use the `upgradeOptions.schedule` option to specify the update check time in CRON format.

    The following example sets the midnight update checks with the official
    Percona’s Version Service:

    ```yaml
    spec:
      updateStrategy: SmartUpdate
      upgradeOptions:
        apply: Recommended
        versionServiceEndpoint: https://check.percona.com
        schedule: "0 0 * * *"
    ...
    ```

    !!! note

        You can force an immediate upgrade by changing the schedule to
        `* * * * *` (continuously check and upgrade) and changing it back to
        another more conservative schedule when the upgrade is complete.

6. Don't forget to apply your changes to the Custom Resource in the usual way:

    ``` {.bash data-prompt="$" }
    $ kubectl apply -f deploy/cr.yaml
    ```

## More on upgrade strategies

The recommended way to upgrade your cluster is to use the
*Smart Update strategy*, when the Operator controls how the objects
are updated. Smart Update strategy is on when the `updateStrategy` key in the
[Custom Resource](operator.md) configuration file is set to `SmartUpdate`
(this is the default value and the recommended way for upgrades).

`SmartUpdate` strategy is not just for simplifying
upgrades. Being turned on, it allows to disable automatic upgrades, and still
controls restarting Pods in a proper order for changes triggered by other
events, such as updating a ConfigMap, rotating a password, or changing resource
values. That's why `SmartUpdate` strategy is useful even when you have no plans
to automate upgrades at all.
