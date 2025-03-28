
# Auto Instrumentation via Otel Operator Collector
Reference Documentation: https://opentelemetry.io/docs/


# 1. Create the Cluster using Kind  

    kind create cluster -n otel-operator

# 2. Install Cert Manager via Helm Chart  

    kubectl apply -f https://github.com/cert-manager/cert-manager/releases/download/v1.16.1/cert-manager.yaml
    kubectl wait --for=condition=Available deployments/cert-manager -n cert-manager

# 3. Install Otel Collector and Instrumentation 

    kubectl apply -f manifests/operator.yaml
    kubectl apply -f manifests/collector.yaml
    kubectl apply -f manifests/instrumentation.yaml

# 4. Build three Docker Images (python, nodejs, dotnet)

    docker build -f applications/python/Dockerfile -t rolldice-python:latest applications/python
    docker build -f applications/nodejs/Dockerfile -t rolldice-nodejs:latest applications/nodejs
    docker build -f applications/dotnet/Dockerfile -t rolldice-dotnet:latest applications/dotnet

# 5. Load all three Images to Minikube Cluster  

    kind load docker-image -n otel-operator rolldice-python:latest
    kind load docker-image -n otel-operator rolldice-nodejs:latest
    kind load docker-image -n otel-operator rolldice-dotnet:latest

    kubectl apply -f manifests/applications/python.yaml
    kubectl apply -f manifests/applications/nodejs.yaml
    kubectl apply -f manifests/applications/dotnet.yaml

# 6. Install Petclinic Java Springboot application  

    kubectl apply -f manifests/applications/petclinic.yaml

# 7. Validate the Metrics on Grafana  

To Search all of the time series data points grouping by job  in query  

    count({__name__=~".+"}) by (job)

# 8. Test all the applications by generating traffice

Generate the traffic  

    kubectl port-forward pod/rolldice-python-7bbd9b4757-5gwvt 1111:8082
    http://127.0.0.1:1111/rolldice

    kubectl port-forward svc/petclinic 1111:80
    http://127.0.0.1:1111

Import the Beyla Dashboard

    https://grafana.com/grafana/dashboards/19923-beyla-red-metrics/