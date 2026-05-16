[https://istio.io/latest/docs/ambient/getting-started/

# Agregar Istio a los repositorios de Helm

    helm repo add istio https://istio-release.storage.googleapis.com/charts
    helm repo update
________________

# Instalar Istio Base

    helm upgrade --install istio-base istio/base -n istio-system --create-namespace --version 1.29.2 --wait -f stack/istio/istio-base-values.yaml

________________
# Instalar CRD de Gateway API (todo en una sola linea)

    kc get crd gateways.gateway.networking.k8s.io &> /dev/null || \
    kc apply --server-side -f https://github.com/kubernetes-sigs/gateway-api/releases/download/v1.5.1/experimental-install.yaml
________________

# Instalar Istiod

    helm upgrade --install istiod istio/istiod --namespace istio-system --version 1.29.2 --wait -f stack/istio/istiod-values.yaml
___________
# Instalar Istio CNI

Pone un CNI en cada node para redirigir el tráfico a los sidecars. Es necesario instalarlo tanto para el modo ambient como para el modo de dataplane tradicional.

    helm upgrade --install istio-cni istio/cni -n istio-system --version 1.29.2 --wait -f stack/istio/istio-cni-values.yaml
____________
# Instalar Istio Ztunnel

Trabajan en cada node junto con los CNI, y se encargan de redirigir el tráfico a los sidecars. No es necesario instalarlo si se va a usar el modo de dataplane tradicional, pero es un requisito para el modo ambient.
    
    helm upgrade --install ztunnel istio/ztunnel -n istio-system --version 1.29.2 --wait -f stack/istio/ztunnel-values.yaml
___________
# Configurar el namespace default para usar el modo ambient

    kc label namespace default istio.io/dataplane-mode=ambient
    kc label ns default istio.io/use-waypoint=waypoint
    istioctl ztunnel-config service
_______________

# Instalar el gateway de istio
Primero desistalar el balanceador de carga externo (k8s svc), y luego instalar el gateway de istio
    
    kc delete svc frontend-external
    kc apply -f stack/istio/gateway-ingress.yaml
__________________

# Habilitar la telemetría de Istio
    
    kc apply -f telemetry.yaml
______________

ki get po -o wide
istioctl proxy-status
k get gatewayclass
k delete svc frontend-external
___________

# En caso error de CNI
En fedora wsl
    
    ulimit -n
    cat /proc/sys/fs/inotify/max_user_watches
    cat /proc/sys/fs/inotify/max_user_instances

    sudo sysctl -w fs.inotify.max_user_watches=524288
    sudo sysctl -w fs.inotify.max_user_instances=1024
    sudo sysctl -w fs.file-max=2097152
    ulimit -n 65535

    kubectl delete pods -n istio-system -l k8s-app=istio-cni-node
](https://istio.io/latest/docs/ambient/getting-started/

# Agregar Istio a los repositorios de Helm

    helm repo add istio https://istio-release.storage.googleapis.com/charts
    helm repo update
________________

# Instalar Istio Base

    helm upgrade --install istio-base istio/base -n istio-system --create-namespace --version 1.29.2 --wait -f stack/istio/istio-base-values.yaml

________________
# Instalar CRD de Gateway API (todo en una sola linea)

    kc get crd gateways.gateway.networking.k8s.io &> /dev/null || \
    kc apply --server-side -f https://github.com/kubernetes-sigs/gateway-api/releases/download/v1.5.1/experimental-install.yaml
________________

# Instalar Istiod

    helm upgrade --install istiod istio/istiod --namespace istio-system --version 1.29.2 --wait -f stack/istio/istiod-values.yaml
___________
# Instalar Istio CNI

Pone un CNI en cada node para redirigir el tráfico a los sidecars. Es necesario instalarlo tanto para el modo ambient como para el modo de dataplane tradicional.

    helm upgrade --install istio-cni istio/cni -n istio-system --version 1.29.2 --wait -f stack/istio/istio-cni-values.yaml
____________
# Instalar Istio Ztunnel

Trabajan en cada node junto con los CNI, y se encargan de redirigir el tráfico a los sidecars. No es necesario instalarlo si se va a usar el modo de dataplane tradicional, pero es un requisito para el modo ambient.
    
    helm upgrade --install ztunnel istio/ztunnel -n istio-system --version 1.29.2 --wait -f stack/istio/ztunnel-values.yaml
___________
# Configurar el namespace default para usar el modo ambient

    kc label namespace default istio.io/dataplane-mode=ambient
    kc label ns default istio.io/use-waypoint=waypoint
    istioctl ztunnel-config service
_______________

# Instalar el gateway de istio
Primero desistalar el balanceador de carga externo (k8s svc), y luego instalar el gateway de istio
    
    kc delete svc frontend-external
    kc apply -f stack/istio/gateway-ingress.yaml
__________________

# Habilitar la telemetría de Istio
    
    kc apply -f telemetry.yaml
______________

ki get po -o wide
istioctl proxy-status
k get gatewayclass
k delete svc frontend-external
___________

# En caso error de CNI
En fedora wsl
    
    ulimit -n
    cat /proc/sys/fs/inotify/max_user_watches
    cat /proc/sys/fs/inotify/max_user_instances

    sudo sysctl -w fs.inotify.max_user_watches=524288
    sudo sysctl -w fs.inotify.max_user_instances=1024
    sudo sysctl -w fs.file-max=2097152
    ulimit -n 65535
    kubectl delete pods -n istio-system -l k8s-app=istio-cni-node

# En caso error de Ztunnel
En fedora wsl

    helm upgrade --install istio-cni istio/cni \
      -n istio-system \
      --version 1.29.2 \
      --set profile=ambient \
      --set global.platform=k3s \
      --set ambient.enabled=true

Luego reiniciar todo

    kubectl rollout restart ds/istio-cni-node -n istio-system
    kubectl rollout restart ds/ztunnel -n istio-system
    kubectl get pods -n istio-system -w)
