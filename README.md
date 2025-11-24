# copyparty-chart

A lightweight Helm chart for [copyparty](https://github.com/9001/copyparty) that prioritizes
transparency and flexibility.

Linted with [`kube-score`](https://github.com/zegl/kube-score),
[`kube-linter`](https://github.com/stackrox/kube-linter)
and [`yamllint`](https://github.com/stackrox/kube-linter).
Tested with [`helm-unittest`](https://github.com/helm-unittest/helm-unittest).

## Design Philosophy

### No Magic

This chart may diverge from some common chart conventions.
The goal is to provide a chart that is configurable and understandable.
To achieve this, hidden logic and magic behavior is avoided as much as possible.
But this comes at the cost of convenience:
For example, if you want to change the ports the application is using, you'd have to adapt both
`.Values.containerPorts` and whatever fields are configuring the application
(environment variables at `.Values.app.env` or arguments at `.Values.app.args`,
or config files at `.Values.configMap.data`).

### No Dependencies

This chart does not bundle any databases (e.g., Postgres, Valkey) as a dependency.
While this may seem inconvenient, it keeps the chart lightweight, and ensures users actively design their setup.
Which database should be used? Is one already deployed? Should a new one be deployed? Is it deployed via helm chart or
operator? Which helm chart should be used? Is a managed database service used?
These are all questions that should be answered by the user, not the chart.

### No Network Policies

This chart does not contain templates for K8s objects like `NetworkPolicy`.
While network policies are important for securing workloads,
they are highly dependent on the specific cluster setup and requirements.
It's the users responsibility to define and manage network policies that fit their needs.

## Getting Started

### Deploy the Helm Chart

If the default values in the [`values.yaml`](./values.yaml) fit your needs,
you can deploy the helm chart using this command:

```shell
helm install copyparty oci://ghcr.io/ernail/charts/copyparty \
--namespace copyparty \
--create-namespace
```

### Configure the Helm Chart

Helm provides different ways to [configure helm charts via values](https://helm.sh/docs/helm/helm_install/#synopsis).
A common way is to create your own values file,
which overrides values of the charts default [`values.yaml`](./values.yaml):

```shell
helm install copyparty oci://ghcr.io/ernail/charts/copyparty \
--namespace copyparty \
--create-namespace \
--values values-base.yaml
```

You can also pass in multiple values files. For example if you need seperate configuration for your `dev` environment:

```shell
helm install copyparty oci://ghcr.io/ernail/charts/copyparty \
--namespace copyparty \
--create-namespace \
--values values-base.yaml \
--values values-dev.yaml
```

All configuration options are documented in the [`values.yaml`](./chart/values.yaml).
An example config is available in the [`values.yaml`](./chart/values.yaml).
Example deployments are available in the [`examples`](./examples) directory.

### Key Configuration Options

#### Copyparty Configuration

Using the default values, since the copyparty configuration can contain sensitive information,
the charts expects you to provide a copyparty configuration file as a `Secret` with the name `copyparty-config`
and the key `config.conf`.
You can find an example configuration at `.Values.configMap.data`.

The chart also supports using a ConfigMap to provide the configuration.
To do so, set `.Values.configMap.enabled` to `true` and adapt `.Values.configMap.data` accordingly.
You also have to adapt `.Values.volumes`, `.Values.volumeMounts` and `.Values.app.args` accordingly.

The chart also supports using command line arguments instead of a configuration file.
To do so, remove the volumes from `.Values.volumes` and `.Values.volumeMounts`
and add the arguments to `.Values.app.args`.

#### Metrics

Metrics can be enabled by setting `.Values.metrics.enabled` to `true`.
However, copyparty only allows access to the metrics if the user has `admin` access.
You need to configure copyparty so a user with `admin` access exists that will be used for accessing the metrics.
When enabling the ServiceMonitor via `.Values.metrics.serviceMonitor.enabled`, you have to configure the authentication
of the endpoints to scrape via any supported authentication method of the ServiceMonitor.
For example, you can use basic authentication by setting `.Values.metrics.serviceMonitor.basicAuth`.

#### Miscellaneous

Other important configuration options that should be reviewed are:

- `ingress` - The ingress configuration
- `resources` - The resource requests and limits
- `metrics` - The metrics configuration
- `volumeConfigs` - The volume and persistence settings

## Contributing

Please check the [`CONTRIBUTING.md`](./CONTRIBUTING.md) to learn how to contribute.

## Development

### Installing dependencies

You can install all required dependencies via `Task` and `Homebrew`

```shell
brew install go-task
task install
```

If you'd like to use other tools,
you can find all dependencies and relevant commands in the [`taskfile.yaml`](./taskfile.yaml)

### Rendering the Helm Chart

```shell
task render
```

### Testing the Helm Chart

```shell
task test
```

### Running linters

```shell
task lint
```

### Generating documentation

```shell
task docs
```
