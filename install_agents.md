# Prerequisites

## Technology stack
- [Kubernetes](https://kubernetes.io/releases/) Version 1.18 or later is required for Causely. 
- [Helm](https://github.com/helm/helm/releases)  Version 3.8 or later is required for Causely.
- [Istio](https://istio.io/latest/docs/ops/integrations/prometheus/) Version 1.15 or later is required for Causely. 
- [EBPF](https://ebpf.io/what-is-ebpf/) Alternatively, an eBPF-enabled kernel version 5.4 or later is also supported by Causely. GKE, EKS, and AKS are examples of managed Kubernetes services that have eBPF-enabled kernels by default.

## Observability stack
- [Prometheus](https://github.com/prometheus-operator/kube-prometheus) Causely leverages Prometheus with exporters for Postgres, Redis, Golang mutex and garbage collection metrics, and other custom applications.
- [Datadog](https://docs.datadoghq.com/monitors/) Alternatively, Causely can leverage Datadog monitors for Postgres, Redis, and other integrations.

## Access
- Kubernetes API access is required to install the Causely Helm chart, which includes a deployment and a daemonset.
- Prometheus endpoint access to query time-series metrics OR
- Datadog API access to query monitored applications and monitor state

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
