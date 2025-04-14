
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

# 3. Install Otel Collector and Instrumentation 

    kubectl apply -f manifests/collector2.yaml
    kubectl apply -f manifests/instrumentation.yaml

# 4. Install Petclinic Java Springboot application  

    kubectl apply -f manifests/applications/petclinic.yaml

# 5. Validate the Metrics on Grafana  
To Search all of the time series data points grouping by job  in query  

    count({__name__=~".+"}) by (job)