
# Auto Instrumentation via Otel Operator Collector
Reference Documentation: https://opentelemetry.io/docs/

# 1. Install Cert Manager via Helm Chart  

    kubectl apply -f https://github.com/cert-manager/cert-manager/releases/download/v1.16.1/cert-manager.yaml
    kubectl wait --for=condition=Available deployments/cert-manager -n cert-manager

# 2. Install OpenTelemetry Operator using Helm Chart  

    helm repo add open-telemetry https://open-telemetry.github.io/opentelemetry-helm-charts
    helm upgrade --install my-opentelemetry-operator open-telemetry/opentelemetry-operator \
    --set "manager.collectorImage.repository=otel/opentelemetry-collector-k8s" \
    --set admissionWebhooks.certManager.enabled=true \
    --set admissionWebhooks.autoGenerateCert.enabled=true

# 3. Install Prometheus Node Exporter on Minikube Cluster

    helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
    helm repo update
    helm upgrade --install node-exporter prometheus-community/prometheus-node-exporter

# 4. Configure and Run Alloy Collector on Minikube Cluster

Make sure to update the node_exporter service ip in alloy configuration in configmap.alloy file before creating configmap

    kubectl create configmap --namespace metrics alloy-config "--from-file=config_k8s_metrics_logs_tracing.alloy=./config_k8s_metrics_logs_tracing.alloy"

    helm  upgrade --install --namespace metrics alloy grafana/alloy -f alloy_k8s_app_monitoring.yaml


# 5. Install Instrumentation 

    kubectl apply -f manifests/instrumentation_alloy.yaml

# 6. Install Petclinic Java Springboot application  

    kubectl apply -f manifests/applications/petclinic.yaml

# 7. Validate the Metrics on Grafana  

To Search all of the time series data points grouping by job  in query  

    count({__name__=~".+"}) by (job)