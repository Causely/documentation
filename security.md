# Security Architecture of Causely

This document outlines the security architecture of Causely, with the objective of providing a comprehensive understanding of the platform's security measures.

Causely endeavors to minimize the transfer of data to its SaaS backend and instead aims to retain most of the data within the customer environment. This primarily encompasses the service topology and detected symptoms such as high CPU utilization. The raw source data, such as metrics and logs, are retained in the customer's environment and are not transmitted to the SaaS backend.

## SaaS Backend

Causely does not store or process any sensitive data or personally identifiable information (PII) that may be available in the customer cluster. All data is encrypted both in transit and at rest. Access to the Causely infrastructure is strictly restricted to a select group of personnel responsible for operating the service.

## Mediation

The mediation comprises four components installed into a customer's Kubernetes cluster. None of these components access any sensitive information such as Kubernetes secrets. As a default, all components operate with minimal privileges, including non-root user status, absence of privilege (securityContext.privileged=false), no hostPath mounts, and no access to the Kubernetes API, unless stated otherwise in subsequent sections.

### Agents

Agents are deployed as a Kubernetes Daemonset across all nodes in the cluster to gather node and container level metrics. This necessitates several permissions:

- The container runs as privileged (securityContext.privileged=true)
- The container uses root as user
- It mounts the host filesystem into the container, granting access to the host directly

This is required as the Causely agent leverages eBPF technology, which necessitates privileged access. Additional Kubernetes API permissions are required to collect specific metrics about the node and containers.

``` yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
rules:
  - apiGroups: [""]
    resources: ["services","nodes", "nodes/metrics"]
    verbs: ["get", "watch", "list"]
  - nonResourceURLs:
      - /metrics
      - /metrics/cadvisor
    verbs:
      - get
```

Agents do not establish any outbound connections to the internet or any other service apart from the Mediator and Victoria Metrics. The agents periodically forward the topology and manifestation data to the Mediator, which, in turn, sends it to the Causely SaaS backend for analysis.

### Mediator

The Mediator is installed as a Kubernetes Deployment and is responsible for collecting cluster-level metrics and transmitting the data to the SaaS backend. Specific permissions are necessary to collect cluster-level metrics.

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
rules:
  - apiGroups: ["", "apps", "app.k8s.io", "batch"]
    resources: ["pods", "nodes", "services", "events", "deployments", "replicasets", replicationcontrollers, "jobs", "cronjobs", "statefulsets", "daemonsets"]
    verbs: ["get", "watch", "list"]
```

and

TODO: remove again

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
rules:
  - apiGroups: ["apps"]
    resources: ["deployments", "replicasets", "daemonsets"]
    verbs: ["update"]
  - apiGroups: [""]
    resources: ["pods/log"]
    verbs: ["get"]
```

### Executor

The Executor is responsible for executing remediation actions within the cluster, and the Kubernetes ServiceAccount assigns the `cluster-admin` role to it.

### Victoria Metrics

Victoria Metrics is a timeseries data used by the agents and mediator to store additional timeseries data.

## Conclusion

Causely places a premium on security, and this document provides an overview of its security measures. These measures ensure that the platform operates optimally and securely while minimizing any risks to customer data.
