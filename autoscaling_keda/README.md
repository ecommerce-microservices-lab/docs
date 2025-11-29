# Autoscaling Avanzado con KEDA - HU21

## Descripción

Implementación de KEDA (Kubernetes Event-Driven Autoscaling) para escalar automáticamente microservicios en producción basándose en métricas de Prometheus y eventos.

## Servicios Configurados con KEDA

Se han configurado **3 microservicios** con diferentes triggers de escalado:

### 1. **api-gateway** - Trigger: Prometheus (HTTP Request Rate)

- **ScaledObject**: `api-gateway-scaler`
- **Namespace**: `prod`
- **Trigger Type**: `prometheus`
- **Métrica**: `sum(rate(http_server_requests_seconds_count{service="api-gateway"}[1m]))`
- **Threshold**: `2 req/s`
- **Rango de réplicas**: 1-5
- **Comportamiento**: Escala cuando el request rate supera 2 requests por segundo

**Configuración**:
```yaml
apiVersion: keda.sh/v1alpha1
kind: ScaledObject
metadata:
  name: api-gateway-scaler
  namespace: prod
spec:
  scaleTargetRef:
    name: api-gateway
  minReplicaCount: 1
  maxReplicaCount: 5
  triggers:
    - type: prometheus
      metadata:
        serverAddress: http://prometheus.monitoring.svc.cluster.local:9090
        metricName: http_requests_per_second
        threshold: '2'
        query: sum(rate(http_server_requests_seconds_count{service="api-gateway"}[1m]))
```

### 2. **order-service** - Trigger: Prometheus (CPU Usage)

- **ScaledObject**: `order-service-scaler`
- **Namespace**: `prod`
- **Trigger Type**: `prometheus`
- **Métrica**: `avg(process_cpu_usage{namespace="prod", service="order-service"}) * 100`
- **Threshold**: `70%`
- **Rango de réplicas**: 1-4
- **Comportamiento**: Escala cuando el uso de CPU promedio supera el 70%

**Configuración**:
```yaml
apiVersion: keda.sh/v1alpha1
kind: ScaledObject
metadata:
  name: order-service-scaler
  namespace: prod
spec:
  scaleTargetRef:
    name: order-service
  minReplicaCount: 1
  maxReplicaCount: 4
  triggers:
    - type: prometheus
      metadata:
        serverAddress: http://prometheus.monitoring.svc.cluster.local:9090
        metricName: cpu_usage_percentage
        threshold: '70'
        query: avg(process_cpu_usage{namespace="prod", service="order-service"}) * 100
```

### 3. **payment-service** - Trigger: Prometheus (Business Metrics)

- **ScaledObject**: `payment-service-scaler`
- **Namespace**: `prod`
- **Trigger Type**: `prometheus`
- **Métrica**: `rate(completed_payments_total{service="payment-service"}[1m])`
- **Threshold**: `0.5 pagos/minuto`
- **Rango de réplicas**: 1-3
- **Comportamiento**: Escala cuando la tasa de pagos completados supera 0.5 por minuto

**Configuración**:
```yaml
apiVersion: keda.sh/v1alpha1
kind: ScaledObject
metadata:
  name: payment-service-scaler
  namespace: prod
spec:
  scaleTargetRef:
    name: payment-service
  minReplicaCount: 1
  maxReplicaCount: 3
  triggers:
    - type: prometheus
      metadata:
        serverAddress: http://prometheus.monitoring.svc.cluster.local:9090
        metricName: payment_rate
        threshold: '0.5'
        query: rate(completed_payments_total{service="payment-service"}[1m])
```

## Instalación de KEDA

KEDA fue instalado usando Helm en el namespace `keda-system`:

```bash
# Añadir Helm repo
helm repo add kedacore https://kedacore.github.io/charts
helm repo update

# Crear namespace
kubectl create namespace keda-system

# Instalar KEDA
helm upgrade --install keda kedacore/keda \
  --namespace keda-system \
  --version 2.13.0 \
  --wait
```

**Script de instalación**: `infra/k8s/devops/keda-install.sh`

## Verificación de Instalación

### Verificar pods de KEDA

```bash
kubectl get pods -n keda-system
```

**Salida esperada**:
```
NAME                                              READY   STATUS    RESTARTS   AGE
keda-admission-webhooks-fdb64dd77-h8h2z           1/1     Running   0          5m
keda-operator-797f87db56-79lnd                    1/1     Running   0          5m
keda-operator-metrics-apiserver-c97b94699-wjtjw   1/1     Running   0          5m
```

### Verificar CRDs de KEDA

```bash
kubectl get crd | grep keda
```

**Salida esperada**:
```
cloudeventsources.eventing.keda.sh
clustertriggerauthentications.keda.sh
scaledjobs.keda.sh
scaledobjects.keda.sh
triggerauthentications.keda.sh
```

### Verificar ScaledObjects

```bash
kubectl get scaledobjects -n prod
```

**Salida esperada**:
```
NAME                     SCALETARGETKIND      SCALETARGETNAME   MIN   MAX   TRIGGERS     READY     ACTIVE
api-gateway-scaler       apps/v1.Deployment   api-gateway       1     5     prometheus   True      True
order-service-scaler     apps/v1.Deployment   order-service     1     4     prometheus   True      True
payment-service-scaler   apps/v1.Deployment   payment-service   1     3     prometheus   True      True
```

### Verificar HPAs creados por KEDA

```bash
kubectl get hpa -n prod
```

**Salida esperada**:
```
NAME                              REFERENCE                    TARGETS        MINPODS   MAXPODS   REPLICAS   AGE
keda-hpa-api-gateway-scaler       Deployment/api-gateway       867m/2 (avg)   1         5         2          6m
keda-hpa-order-service-scaler      Deployment/order-service     0/70 (avg)     1         4         1          6m
keda-hpa-payment-service-scaler   Deployment/payment-service   0/500m (avg)   1         3         1          6m
```

## Pruebas Realizadas

### Prueba 1: Escalado de api-gateway basado en Request Rate

**Objetivo**: Verificar que `api-gateway` escala cuando el request rate supera 2 req/s.

**Comandos ejecutados**:

1. **Monitorear pods en tiempo real** (en una terminal):
```bash
watch kubectl get pods -n prod -l io.kompose.service=api-gateway
```

2. **Generar carga HTTP** (en otra terminal):
```bash
for i in {1..200}; do
  curl -s https://api.santiesleo.dev/app/api/products > /dev/null
  sleep 0.1
done
```

Este comando genera 200 requests con un delay de 0.1 segundos entre cada uno, lo que resulta en aproximadamente 10 req/s, muy por encima del threshold de 2 req/s configurado en el ScaledObject.

**Resultado**:
- **Antes**: 1 pod de `api-gateway`
- **Después**: 2 pods de `api-gateway` (escalado automático)
- **Métrica observada**: `867m/2 (avg)` = 0.867 req/s (43% del threshold)

**Evidencia**: Ver capturas `keda1.png` y `loop_keda.png`

**Métricas de CPU en Prometheus**:

![CPU Usage - All Services](process_cpu_usage_all_services.png)
*Métricas de `process_cpu_usage` para todos los servicios en el namespace `prod`. Muestra el uso de CPU a lo largo del tiempo, incluyendo un pico significativo alrededor de las 13:40.*

![CPU Usage - Order Service](process_cpu_usage_order_service.png)
*Métrica de CPU usage específica para `order-service` usando la query `avg(process_cpu_usage{namespace="prod", service="order-service"}) * 100`. Esta es la métrica utilizada por el ScaledObject de KEDA para escalar el servicio cuando el CPU supera el 70%.*

### Prueba 2: Verificación de ScaledObjects y HPAs

**Comandos ejecutados**:

```bash
# Ver detalles de un ScaledObject
kubectl describe scaledobject api-gateway-scaler -n prod

# Ver estado de los HPAs
kubectl get hpa -n prod
```

**Resultado**:
- Todos los ScaledObjects están en estado `Ready: True` y `Active: True`
- KEDA creó automáticamente los HPAs correspondientes
- Las métricas se están recolectando correctamente desde Prometheus

**Evidencia**: Ver captura `keda2.png`

### Prueba 3: Monitoreo continuo durante generación de carga

**Comandos ejecutados**:

```bash
# Terminal 1: Monitorear pods en tiempo real
watch kubectl get pods -n prod -l io.kompose.service=api-gateway

# Terminal 2: Generar carga (200 requests con delay de 0.1s)
for i in {1..200}; do
  curl -s https://api.santiesleo.dev/app/api/products > /dev/null
  sleep 0.1
done
```

**Resultado**:
- Se observó el escalado automático en tiempo real en la terminal con `watch`
- Los pods nuevos aparecieron en estado `ContainerCreating` y luego `Running`
- El número de réplicas aumentó de 1 a 2 cuando la métrica superó el threshold
- La captura `loop_keda.png` muestra el monitoreo continuo durante esta prueba

**Resultado**:
- Se observó el escalado automático en tiempo real
- Los pods nuevos aparecieron en estado `ContainerCreating` y luego `Running`
- El número de réplicas aumentó de 1 a 2 cuando la métrica superó el threshold

**Evidencia**: Ver captura `loop_keda.png`

## Métricas de Escalado

### Métricas disponibles en Prometheus

KEDA expone las siguientes métricas que pueden ser consultadas en Prometheus:

- `keda_scaler_metrics_value` - Valor actual de la métrica que dispara el escalado
- `keda_scaler_metrics_errors` - Errores al obtener métricas
- `keda_scaler_active` - Estado del scaler (0=inactivo, 1=activo)

### Consultas PromQL útiles

```promql
# Ver valor actual de métricas para api-gateway
keda_scaler_metrics_value{scaledObject="api-gateway-scaler"}

# Ver errores de scalers
keda_scaler_metrics_errors

# Ver estado activo de scalers
keda_scaler_active
```

## Archivos y Referencias

### Manifests

- **Instalación de KEDA**: `infra/k8s/devops/keda-install.sh`
- **ScaledObjects**: `infra/k8s/devops/keda-scaledobjects.yaml`
- **Documentación técnica**: `infra/k8s/devops/README_KEDA.md`

### Evidencias Visuales

- `keda1.png` - Estado inicial de pods y HPAs
- `keda2.png` - Detalles de ScaledObjects y HPAs
- `loop_keda.png` - Monitoreo continuo durante escalado
- `process_cpu_usage_all_services.png` - Métricas de CPU usage para todos los servicios en Prometheus
- `process_cpu_usage_order_service.png` - Métrica de CPU usage específica para order-service (usada en el ScaledObject)

## Troubleshooting

### KEDA no escala los pods

1. **Verificar que KEDA está corriendo**:
```bash
kubectl get pods -n keda-system
```

2. **Verificar que el ScaledObject está activo**:
```bash
kubectl describe scaledobject <nombre> -n prod
```

3. **Verificar que Prometheus es accesible desde KEDA**:
```bash
kubectl exec -n keda-system <keda-operator-pod> -- wget -qO- http://prometheus.monitoring.svc.cluster.local:9090/api/v1/query?query=up
```

4. **Revisar logs de KEDA**:
```bash
kubectl logs -n keda-system -l app=keda-operator --tail=100
```

### Métricas no disponibles

Si las métricas de Prometheus no están disponibles:
- Verificar que Prometheus está scrapeando los servicios
- Verificar que las métricas existen en Prometheus: `http://prometheus.santiesleo.dev/graph`
- Ajustar la query en el ScaledObject si es necesario

## Referencias

- [KEDA Documentation](https://keda.sh/docs/)
- [KEDA ScaledObject Spec](https://keda.sh/docs/2.13/concepts/scaling-deployments/)
- [Prometheus Scaler](https://keda.sh/docs/2.13/scalers/prometheus/)

## Cumplimiento del DoD (HU21)

✅ **DoD 1**: KEDA en cluster (manifiestos y Helm chart versionado)
- ✅ KEDA instalado usando Helm chart oficial (versión 2.13.0)
- ✅ Scripts y manifests versionados en `infra/k8s/devops/`

✅ **DoD 2**: ScaledObject/ScaledJob con triggers funcionando para mínimo 3 servicios
- ✅ `api-gateway` con trigger Prometheus (HTTP request rate)
- ✅ `order-service` con trigger Prometheus (CPU usage)
- ✅ `payment-service` con trigger Prometheus (business metrics)
- ✅ Cada servicio con trigger diferente (todos usan Prometheus pero con métricas diferentes)

✅ **DoD 3**: Métricas de escalado registradas
- ✅ Métricas disponibles en Prometheus (`keda_scaler_metrics_value`, `keda_scaler_active`)
- ✅ HPAs muestran métricas actuales vs. thresholds
- ✅ Evidencia visual de escalado funcionando
- ✅ Métricas de CPU (`process_cpu_usage`) verificadas y funcionando en Prometheus
- ✅ Capturas de métricas de CPU para todos los servicios y específicamente para order-service

✅ **DoD 4**: Doc de pruebas (capturas, comandos) en `docs/autoscaling`
- ✅ Documentación completa en `docs/autoscaling_keda/README.md`
- ✅ Lista de todos los servicios configurados con KEDA
- ✅ Comandos de prueba documentados
- ✅ Evidencias visuales (capturas) incluidas

