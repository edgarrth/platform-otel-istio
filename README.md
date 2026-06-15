cert-manager: (used by otel installation)
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

Grafana Agent
---------------
    kubectl -n prometheus create secret generic grafana-pdc-agent \
     --from-literal="token=1212==" \
     --from-literal="hosted-grafana-id=121212121" \
     --from-literal="cluster=prod-us-east-0"
    
    kubectl -n prometheus apply -f https://raw.githubusercontent.com/grafana/pdc-agent/main/production/kubernetes/pdc-agent-deployment.yaml


Auto Instrumentation
--------------
    kubectl apply -f stack/otel/autoinstrumentation.yml

Instalar Servicios para instrumentalizar
--------------
    kubectl apply -f k8s/kubernetes-manifests_v2.yaml


Prometheus CRDs
-------------
    helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
    helm repo update
    helm upgrade --install prometheus-operator-crds prometheus-community/prometheus-operator-crds -n monitoring --create-namespace
    K8s-monitoring:
    helm repo add grafana https://grafana.github.io/helm-charts
    helm repo update
    helm upgrade --install k8s-monitoring grafana/k8s-monitoring -n grafana-agent --create-namespace --version 4.0.3 -f stack/k8s-monitoring/values.yaml
    Prometheus:
    helm repo add prometheus-community https://prometheus-community.github.io/helm-charts helm repo update helm upgrade --install prometheus prometheus-community/prometheus -n prometheus --create-namespace --version 29.6.0 -f stack/prometheus/values-server.yaml

# Istio:
https://istio.io/latest/docs/ambient/install/helm/

helm repo add istio https://istio-release.storage.googleapis.com/charts
helm repo update
helm upgrade --install istio-base istio/base -n istio-system --create-namespace --version 1.29.2 --wait -f stack/istio/istio-base-values.yaml

k get crd gateways.gateway.networking.k8s.io &> /dev/null || \
k apply --server-side -f https://github.com/kubernetes-sigs/gateway-api/releases/download/v1.5.1/experimental-install.yaml

helm upgrade --install istiod istio/istiod --namespace istio-system --version 1.29.2 --wait -f stack/istio/istiod-values.yaml

# traffic redirection between pods and the ztunnel node proxy
# compatible con modo sidecar
helm upgrade --install istio-cni istio/cni -n istio-system --version 1.29.2 --wait -f stack/istio/istio-cni-values.yaml

sidecar:
--------
    kubectl label ns default istio.io/dataplane-mode-
    kubectl label ns default istio.io/use-waypoint-
    kubectl label ns default istio.io/use-waypoint-namespace-

    kubectl label namespace default istio-injection=enabled
    -> revisar anotaciones entre opentelemetry & istio (del ns inyecta init container & sidecar)
    kubectl -n prometheus port-forward svc/prometheus-server 9090:80 k label namespace default istio-injection=enabled k logs -f frontend-7d5546bff7-jh57f -c istio-proxy

    kubectl run nginx --image nginx k create ns test k -n test nginx --image nginx k exec -it nginx -- bash curl -I https://api.ipify.org curl -i https://api.ipify.org?format=json k -n test expose pod nginx --port=80 --target-port=80 --name=nginx
