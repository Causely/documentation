# Prerequisites

## Technology stack
- [Kubernetes](https://kubernetes.io/releases/) Version 1.18 or later. 
- [Helm](https://github.com/helm/helm/releases) Version 3.8 or later.
- [Istio](https://istio.io/latest/docs/ops/integrations/prometheus/) Version 1.15 or later. 

## Observability stack
- [Prometheus](https://github.com/prometheus-operator/kube-prometheus) Causely leverages Prometheus to query time-series metrics from your applications and infrastructure with exporters for Kafka, MongoDB, MySQL, Postgres, RabbitMQ, Redis along with Java, Python, Golang mutex and garbage collection metrics, and other custom applications.
- [OpenTelemetry](https://opentelemetry.io/docs/concepts/signals/traces/) Causely leverages OpenTelemetry traces to discover service dependencies and monitor sync and async communication signals
- [eBPF](https://grafana.com/oss/beyla-ebpf/) Linux kernel version 5.4 or later is also supported by Causely. GKE, EKS, and AKS are examples of managed Kubernetes services that have eBPF-enabled kernels by default.
- [Odigos](https://docs.odigos.io/setup/installation) Causely can leverage Odigos traces to discover service dependencies and monitor sync and async communication signals.
- [Groundcover](https://www.groundcover.com/product/application-performance-monitoring) Causely can leverage Groundcover instrumentation for http, grpc, database, messaging and other service dependencies.
- [Datadog](https://docs.datadoghq.com/monitors/) Causely can leverage Datadog monitors for Postgres, Redis, and other integrations.
- AWS, GCP, and Azure managed services are also supported by Causely.

## Access
- Kubernetes API access is required to install the Causely Helm chart, which includes a deployment and a daemonset.
- Prometheus endpoint access to query time-series metrics OR
- OpenTelemetry pipeline to send traces and metrics to Causely
- Datadog API access to query monitored applications and monitor state

# Install Agents

To install the Causely agent, please download the Causely CLI tool using the following command:

```shell
bash -c "$(curl -fksSL https://www.causely.io/install.sh)"
```

Once the CLI tool is downloaded, log in with your credentials and install the agents into your cluster. The Causely agents will be installed in your current kubectl context using the following commands:

```shell
causely auth login

causely agent install --cluster-name <cluster-name>
```

If you require custom configuration options, you may create a values.yaml file and specify it as part of the install command using the following command:

```shell
causely agent install --cluster-name <cluster-name> --values ./causely-values.yaml
```

Please refer to the example [causely-values.yaml](./causely-values.yaml) for all available configuration options.

## Prerequisites for Deploying on Openshift
If you are deploying on Openshift, you need to use the group id from the uid-range assigned to the project:
````bash
oc get ns causely -o yaml|grep uid-range
    openshift.io/sa.scc.uid-range: 1000630000/10000
````
and include in the causely-values.yaml file:
````
global:
  securityContext:
    fsGroup: 1000630000
````

Or you can change the security context of the project to the 'anyuid' SCC
````bash
oc adm policy add-scc-to-group anyuid system:serviceaccounts:causely
````

In both cases, you need to assign the 'privileged' SCC to the 'causely-agent' service account used by the Causely agents:
````bash
oc adm policy add-scc-to-user privileged -z causely-agent -n causely
````

## Using a custom StorageClass instead of the default one

If you are deploying into a cluster, where there is no default StorageClass defined, you can
specify the StorageClass to use for persistent volumes:
````
global:
  storageClass: ocs-storagecluster-ceph-rbd
````
Alternatively you can annotate a default StorageClass:

````
kubectl patch storageclass ocs-storagecluster-ceph-rbd -p '{"metadata": {"annotations":{"storageclass.kubernetes.io/is-default-class":"true"}}}'
````

## Configure Datasources

To ensure that Causely can detect a wide range of root causes in your environment, it is recommended that you configure additional data sources if they are available.

### OpenTelemetry Traces

To enable OpenTelemetry as a data source, please install an opentelemtry-collector using this example `opentelemetry-values.yaml` file:

```shell 
helm repo add open-telemetry https://open-telemetry.github.io/opentelemetry-helm-charts
helm install opentelemetry-collector open-telemetry/opentelemetry-collector --values ./opentelemetry-values.yaml
```

### Odigos

To install Odigos, please refer to the [Odigos documentation](https://docs.odigos.io/setup/installation)

To send Metrics/Traces to Causely, you need to configure the Causely URL in the Odigos UI. This destination is for the Causely Mediator Service, so you will need to have a Causely instance running and accessible from the k8s cluster running odigos.

The endpoint URL is the combined <protocol>://<hostname>:<port> to access your Causely Mediator service. For example: http://mediator.causely:4317

    Protocol should be http; using https or omitting it will automatically be converted to http
    Hostname should typically follow the format: mediator.<namespace>
        namespace is the k8s namespace where the Causely Mediator service is deployed
    Default port is 4317; if no port is specified, it will be appended automatically

Please refer to the example [odigos-destination.yaml](./odigos-destination.yaml).

### Prometheus

To enable Prometheus as a data source, please add the following section to your `values.yaml` file:

```yaml
scrapers:
  prometheus:
    enabled: true
    endpoint: http://prometheus-operatored.monitoring.svc.cluster.local:9090
```

Causely will automatically pick up metrics from supported exporters.

Supported Exporters:

- [Kafka](https://github.com/prometheus-community/helm-charts/tree/main/charts/prometheus-kafka-exporter)
- [MongoDB](https://github.com/prometheus-community/helm-charts/tree/main/charts/prometheus-mongodb-exporter)
- [MySQL](https://github.com/prometheus-community/helm-charts/tree/main/charts/prometheus-mysql-exporter)
- [Postgresql](https://github.com/prometheus-community/helm-charts/tree/main/charts/prometheus-postgres-exporter)
- [RabbitMQ](https://github.com/prometheus-community/helm-charts/tree/main/charts/prometheus-rabbitmq-exporter)
- [Redis](https://github.com/prometheus-community/helm-charts/tree/main/charts/prometheus-redis-exporter)

### Istio - Prometheus

The Istio integration works by scraping the Istio relevant data from a Prometheus server.

To enable Istio as a data source, please add the following section to your values.yaml file:

```yaml
scrapers:
  istio:
    enabled: true
    prometheus:
      endpoint: http://prometheus.istio-system.svc.cluster.local:9090
```

Please ensure that the Prometheus instance scrapes all your Istio sidecars to capture all the service-to-service communication metrics.

### Istio - w/o Prometheus

The Istio integration works by scraping the Istio relevant data directly from the istio sidecars.

To enable Istio as a data source, please add the following section to your values.yaml file:

```yaml
scrapers:
  istiosidecar:
    enabled: true
```

### Datadog

To enable Datadog as a data source, please add the following section to your `values.yaml` file:

```yaml
scrapers:
  datadog:
    enabled: true
    org: Customer
    api_key: YYY
    app_key: ZZZ
```

### Groundcover

To enable Groundcover as a data source, please use the following additional [helm values](./groundcover-values.yaml) to  your groundcover install:

```shell
helm install groundcover groundcover/groundcover --values ./groundcover-values.yaml
```

### AWS Managed Services

```yaml 
scrapers:
  aws:
    enabled: true
    secretName: aws-secret
```

Please refer to the example [aws-secret.yaml](./aws-secret.yaml).

### GCP Managed Services

```yaml 
scrapers:
  gcp:
    enabled: true
    projects:
      - id: "playground-123456"
        secretName: gcp-credentials
```

Please refer to the example [gcp-credentials.yaml](./gcp-credentials.yaml).

### Azure Managed Services

```yaml 
scrapers:
  azure:
    enabled: true
    subscriptions:
      - secretName: spn-credentials
        subscriptionId: 00000000-0000-0000-0000-000000000000
```

Please refer to the example [spn-credentials.yaml](./spn-credentials.yaml).
