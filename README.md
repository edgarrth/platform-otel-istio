cert-manager: (used by jaeger or otel installation, reason: grpc)
-------------
    helm repo add jetstack https://charts.jetstack.io
    helm repo update
    helm upgrade cert-manager jetstack/cert-manager --install -n kube-system --version 1.20.2 -f stack/cert-manager/values-cert-manager.yml

Open Telemetry
--------------
    helm repo add open-telemetry https://open-telemetry.github.io/opentelemetry-helm-charts
    helm repo update
    helm upgrade opentelemetry-operator open-telemetry/opentelemetry-operator --version 0.110.0 --install -n operators -f stack/otel/values-operator.yml --create-namespace

Prometheus
--------------
    helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
    helm repo update
    helm upgrade --install prometheus-operator-crds prometheus-community/prometheus-operator-crds -n monitoring --create-namespace

Grafana
--------------
    helm repo add grafana https://grafana.github.io/helm-charts
    helm repo update
    helm upgrade --install k8s-monitoring grafana/k8s-monitoring -n grafana-agent --create-namespace --version 4.0.3 -f stack/otel/values-k8s-monitoring.yaml

Auto Instrumentation
--------------
    kubectl apply -f stack/otel/autoinstrumentation.yml

Instalar Servicios para instrumentalizar
--------------
    kubectl apply -f k8s/kubernetes-manifests_v2.yaml
