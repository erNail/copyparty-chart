# copyparty-chart

A lightweight Helm chart for [copyparty](https://github.com/9001/copyparty) that prioritizes
transparency and flexibility.

Linted with [`kube-score`](https://github.com/zegl/kube-score),
[`kube-linter`](https://github.com/stackrox/kube-linter)
and [`yamllint`](https://github.com/stackrox/kube-linter).
Tested with [`helm-unittest`](https://github.com/helm-unittest/helm-unittest).

## Design Philosophy

### Transparency and Flexibility over Abstraction and Convenience

This chart may diverge from some common chart conventions.
It favors transparency and flexibility over abstraction and convenience:

- **Values-Driven:** The goal is to provide a chart that can be understood just by looking at the values file.
  There should be no need to look into the templates to understand what the chart does and how it works.
- **Minimal Abstraction:** The chart avoids hidden logic and magic behavior as much as possible.
  A lot of what you see in the values file is what gets rendered in the templates.
- **Flexibility over Convenience:** The chart is designed to be flexible and understandable.
  But this comes at the cost of convenience.
  For example, if you want to use arguments instead of a config file mounted from a secret,
  you can do so via editing the values file. But you'd have to make edits in multiple places.
  You'd have to set `.Values.configMap.enabled` to `false`, add the arguments to `.Values.app.args`,
  and remove the ConfigMap volumes from `.Values.volumes` and `.Values.volumeMounts`.

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

### Key Configuration Options

An example config is available in the [`values.yaml`](./chart/values.yaml).
Example deployments are available in the [`examples`](./examples) directory.

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
- `persistence` - The persistence configuration

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
