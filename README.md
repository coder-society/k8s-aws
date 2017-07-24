# k8s-aws
This is a kops wrapper script for creating Kubernetes clusters on AWS and maintaining them. Certain addons are also installed into newly created clusters:

* Heapster
* Kube-Dashboard
* Elasticsearch/Fluentd/Kibana (EFK) logging stack
* [Prometheus Operator](https://github.com/coreos/prometheus-operator) monitoring stack

The cluster is additionally set up for [RBAC authorization](https://kubernetes.io/docs/admin/authorization/rbac/) for heightened security.
