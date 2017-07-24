# k8s-aws
This is a kops wrapper script for creating Kubernetes clusters on AWS and maintaining them.
Certain addons are also installed into newly created clusters:

* Heapster
* Kube-Dashboard
* Elasticsearch/Fluentd/Kibana (EFK) logging stack
* [Prometheus Operator](https://github.com/coreos/prometheus-operator) monitoring stack

The cluster is additionally set up with
[RBAC authorization](https://kubernetes.io/docs/admin/authorization/rbac/) for heightened security.

## Requirements
We use Python 3. To install dependencies, issue the following command at the root of this project:
`pip3 install -r requirements.txt`.

## Secrets
Write your secrets in secrets.yaml, see secrets.yaml.sample.
