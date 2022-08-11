# Monitoring with Prometheus and Grafana:

### Deployment

Create a `prometheus` namespace:
`kubectl create namespace prometheus`

Deploy the `kube-prometheus-stack` Helm Chart from the `prometheus-community` Helm repo:
```
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update
helm install prometheus prometheus-community/kube-prometheus-stack -f values.yaml -n prometheus --set 'grafana.additionalDataSources[0].url=redis://x.x.x.x:6379' --set-file grafana.dashboards.grafana-dashboard-provider.prometheus-postgres-exporter.json=dashboards/prometheus-postgres-exporter.json --set-file grafana.dashboards.grafana-dashboard-provider.prometheus-blackbox-exporter.json=dashboards/prometheus-blackbox-exporter.json --set-file grafana.dashboards.grafana-dashboard-provider.grafana-redis-dashboard.json=dashboards/grafana-redis-dashboard.json --set-file grafana.dashboards.grafana-dashboard-provider.grafana-nginx-dashboard.json=dashboards/grafana-nginx-dashboard.json --set-file grafana.dashboards.grafana-dashboard-provider.prometheus-postgres-exporter-drill-down.json=dashboards/prometheus-postgres-exporter-drill-down.json
```

##### Ingress Setup
- Create authentication credentials for Prometheus dashboard:
`htpasswd -c auth prometheus`

- Create k8s secret using the credentials file auth:
`kubectl create secret generic prometheus-basic-auth --from-file=auth -n prometheus`

- Create the Ingress resource
`kubectl apply -f ingress.yaml`

### Exporters

##### Deploy Prometheus Blackbox Exporter

`helm upgrade --install prometheus-blackbox-exporter prometheus-community/prometheus-blackbox-exporter -n prometheus`

##### Deploy Prometheus Cloudwatch Exporter

`helm upgrade --install prometheus-cloudwatch-exporter prometheus-community/prometheus-cloudwatch-exporter -f prometheus-postgres-exporter/values.yaml -n prometheus`

##### Deploy Prometheus Postgres Exporter

`helm upgrade --install prometheus-postgres-exporter prometheus-community/prometheus-postgres-exporter -f prometheus-postgres-exporter/values.yaml -n prometheus`
