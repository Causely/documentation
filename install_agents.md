# Install Agents

To install the Causely agents, please download the Causely CLI tool using the following command:

```shell
bash -c "$(curl -fksSL https://www.causely.io/install.sh)"
```

Once the CLI tool is downloaded, log in with your credentials and install the agents into your cluster. The Causely agents will be installed in your current kubectl context using the following commands:

```shell
causely auth login

causely agent install
```

If you require custom configuration options, you may create a values.yaml file and specify it as part of the install command using the following command:

```shell
causely agent install --values ./values.yaml
```

Please refer to the example [values.yaml](./values.yaml) for all available configuration options.

## Configure Datasources

To ensure that Causely can detect a wide range of defects in your environment, it is recommended that you configure additional data sources if they are available.

### Prometheus

To enable Prometheus as a data source, please add the following section to your `values.yaml` file:

```yaml
scrapers:
  prometheus:
    endpoint: http://prometheus.monitoring.svc.cluster.local:9090
```

Causely will automatically pick up metrics from supported exporters.

Supported Exporters:

- [Postgresql](https://github.com/prometheus-community/postgres_exporter)

### Istio - Prometheus

The Istio integration works by scraping the Istio relevant data from a Prometheus server.

To enable Istio as a data source, please add the following section to your values.yaml file:

```yaml
scrapers:
  istio:
    prometheus:
      endpoint: http://prometheus.istio-system.svc.cluster.local:9090
```

Please ensure that the Prometheus instance scrapes all your Istio sidecars to capture all the service-to-service communication metrics.
