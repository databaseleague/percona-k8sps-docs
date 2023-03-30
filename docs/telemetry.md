# Telemetry

The Telemetry function enables the Operator gathering and sending basic anonymous data to Percona, which helps us to determine where to focus the development and what is the uptake for each release of Operator.

The following information is gathered:

* ID of the Custom Resource (the `metadata.uid` field)
* Kubernetes version
* Platform (is it Kubernetes or Openshift)
* PMM Version
* Operator version
* Percona Server for MySQL version
* HAProxy version
* Percona XtraBackup version

We do not gather anything that identify a system, but the following thing should be mentioned:
Custom Resource ID is a unique ID generated by Kubernetes for each Custom Resource.

Telemetry is enabled by default and is sent to the [Version Service server](operator.md#upgradeoptions-versionserviceendpoint) when the Operator connects to it at scheduled times to obtain fresh information about version numbers and valid image paths needed for the upgrade.

The landing page for this service, [check.percona.com](https://check.percona.com/), explains what this service is.

You can disable telemetry with a special option when installing the Operator:
edit the `operator.yaml`
before applying it with the `kubectl apply -f deploy/operator.yaml` command.
Open the `operator.yaml` file with your text editor, find the value of the
`DISABLE_TELEMETRY` environment variable and set it to `true`:

  ```yaml
  env:
    ...
    - name: DISABLE_TELEMETRY
      value: "true"
    ...
  ```
