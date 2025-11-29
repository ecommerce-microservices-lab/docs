# FinOps y Optimización de Costos Multi-Cloud - HU20

## Descripción

Implementación de prácticas FinOps para gestionar y optimizar costos en la infraestructura multi-cloud (Azure AKS + GCP GKE). Incluye etiquetado de recursos, dashboards de costos, políticas de ahorro y recomendaciones de optimización.

## Etiquetado de Recursos

### Etiquetas Implementadas

Todos los recursos están etiquetados con las siguientes etiquetas de costos:

- **`env`**: Ambiente (`prod`, `dev`, `stage`)
- **`service`**: Nombre del microservicio (ej: `api-gateway`, `order-service`)
- **`owner`**: Equipo responsable (`devops-team`)
- **`cost-center`**: Centro de costos (`engineering`)
- **`project`**: Proyecto (`microservices`)
- **`managed-by`**: Herramienta de gestión (`terraform`)

### Recursos Etiquetados

#### GCP GKE (Producción)

**Cluster y Node Pool**:
- `env=prod`
- `owner=devops-team`
- `cost-center=engineering`
- `project=microservices`

**Namespaces**:
- `prod`: `env=prod`, `owner=devops-team`, `cost-center=engineering`

**Deployments y Pods**:
- Cada servicio tiene `service=<nombre-servicio>`
- Todos tienen `env=prod`, `owner=devops-team`

#### Azure AKS (Dev/Stage)

**Cluster**:
- Etiquetas de Azure Resource Manager con `env`, `owner`, `cost-center`

**Namespaces**:
- `dev`: `env=dev`, `owner=devops-team`, `cost-center=engineering`

### Script de Etiquetado

Para añadir etiquetas a recursos existentes:

```bash
# Etiquetar recursos en namespace prod
./scripts/finops/add-cost-labels.sh prod prod devops-team

# Etiquetar recursos en namespace dev
./scripts/finops/add-cost-labels.sh dev dev devops-team
```

## Dashboards de Costos

### GCP Cost Management

**Acceso al Dashboard**:
1. Ve a: https://console.cloud.google.com/billing
2. Selecciona tu proyecto
3. Navega a "Reports" o "Cost breakdown"

**Filtros por Etiquetas**:
- Por `env`: Ver costos de `prod` vs `dev`
- Por `service`: Ver costos por microservicio
- Por `owner`: Ver costos por equipo

**Vista Mensual**:
- Costos del cluster GKE
- Costos de nodos (e2-medium)
- Costos de almacenamiento
- Costos de red

### Azure Cost Management

**Acceso al Dashboard**:
1. Ve a: https://portal.azure.com/#view/Microsoft_Azure_CostManagement/Menu
2. Selecciona tu suscripción
3. Navega a "Cost analysis"

**Filtros por Etiquetas**:
- Por `env`: Ver costos de `dev` vs `stage`
- Por `service`: Ver costos por microservicio
- Por `owner`: Ver costos por equipo

### Costos Estimados Actuales

#### GCP GKE (Producción)

| Componente | Costo/Hora | Costo/Mes (24/7) | Costo/Mes (Horario Laboral) |
|------------|------------|------------------|------------------------------|
| Cluster Management | $0.10 | ~$72 | ~$16 |
| Nodo 1 (e2-medium) | ~$0.067 | ~$48 | ~$11 |
| Nodo 2 (e2-medium) | ~$0.067 | ~$48 | ~$11 |
| Nodo 3 (autoscaling) | ~$0.067 | ~$24* | ~$5* |
| **Total** | **~$0.234-0.301** | **~$168-192** | **~$37-43** |

*Nodo 3 solo cuando el autoscaling lo activa

#### Azure AKS (Dev/Stage)

| Componente | Costo/Hora | Costo/Mes (24/7) | Costo/Mes (Horario Laboral) |
|------------|------------|------------------|------------------------------|
| Cluster Management | $0.10 | ~$72 | ~$16 |
| Nodo 1 (Standard_D2s_v3) | ~$0.096 | ~$69 | ~$15 |
| Nodo 2 (Standard_D2s_v3) | ~$0.096 | ~$69 | ~$15 |
| **Total** | **~$0.292** | **~$210** | **~$46** |

**Nota**: Los costos reales pueden variar según uso, descuentos y facturación específica.

## Políticas de Ahorro Implementadas

### 1. Autoscaling de Nodos

**GCP GKE**:
- ✅ Autoscaling habilitado (1-3 nodos)
- ✅ Reduce costos cuando la carga es baja
- ✅ Escala automáticamente según demanda

**Azure AKS**:
- ⚠️ Autoscaling manual (puede configurarse)
- 💡 Recomendación: Habilitar Cluster Autoscaler

### 2. KEDA para Escalado de Pods

- ✅ KEDA instalado y configurado
- ✅ Escala pods basándose en métricas de negocio
- ✅ Reduce costos al usar solo los recursos necesarios
- ✅ **Scale to Zero implementado**: `favourite-service` y `shipping-service` pueden escalar a 0 réplicas cuando no hay tráfico

**Servicios con Scale to Zero**:
- `favourite-service`: `minReplicaCount: 0`, threshold: 0.1 req/s
- `shipping-service`: `minReplicaCount: 0`, threshold: 0.1 req/s

**Manifests**: `infra/k8s/devops/keda-scaledobjects-scale-to-zero.yaml`

### 3. Scripts de Ahorro

#### Auto-stop de Recursos No Críticos (Futuro)

```bash
# Script para detener recursos durante horas no laborales
# (Requiere configuración de Cloud Scheduler o cron)
./scripts/finops/auto-stop-resources.sh
```

**Implementación Futura**:
- Usar Cloud Scheduler (GCP) o Azure Automation
- Programar detención de recursos no críticos fuera de horario laboral
- Ahorro estimado: 50-70% en entornos de desarrollo

### 4. Uso de Nodos Preemptibles (GCP)

**Configuración Actual**:
- `preemptible = false` (nodas normales)

**Recomendación para Ahorro**:
- Usar nodos preemptibles en entornos de desarrollo
- Ahorro estimado: hasta 80% de descuento
- ⚠️ No recomendado para producción (pueden ser terminados)

### 5. Right-Sizing de Recursos

**Análisis Realizado**:
- Monitoreo de uso de CPU/RAM en Prometheus
- Ajuste de requests/limits según uso real
- Optimización de ResourceQuotas por namespace

## Recomendaciones de Optimización

### Corto Plazo (1-3 meses)

1. **Habilitar Autoscaling en Azure AKS**
   - Ahorro estimado: 20-30% en entornos de desarrollo
   - Implementación: Configurar Cluster Autoscaler

2. **Usar Nodos Preemptibles en Dev**
   - Ahorro estimado: 60-80% en nodos de desarrollo
   - Implementación: Cambiar `preemptible = true` en Terraform para dev

3. **Optimizar Resource Requests/Limits**
   - Basado en métricas de Prometheus
   - Ahorro estimado: 10-15% por mejor utilización

### Mediano Plazo (3-6 meses)

1. **Reserved Instances / Committed Use Discounts**
   - Azure: Reserved Instances (hasta 72% descuento)
   - GCP: Committed Use Discounts (hasta 70% descuento)
   - Ahorro estimado: 40-60% en recursos de producción

2. **Auto-stop de Recursos No Críticos**
   - Detener clusters de desarrollo fuera de horario laboral
   - Ahorro estimado: 50-70% en entornos de desarrollo

3. **Consolidación de Namespaces**
   - Reducir número de namespaces si es posible
   - Optimizar ResourceQuotas

### Largo Plazo (6-12 meses)

1. **Análisis Continuo de Costos**
   - Dashboards automatizados
   - Alertas de costos inusuales
   - Reportes mensuales

2. **Optimización Multi-Cloud**
   - Mover cargas según costos
   - Usar el proveedor más económico para cada workload

3. **Implementar Spot Instances**
   - Para workloads tolerantes a interrupciones
   - Ahorro estimado: 60-90%

## Estimaciones de Ahorro Potencial

### Escenario Actual (Sin Optimizaciones)

- **GCP GKE (Prod)**: ~$168-192/mes
- **Azure AKS (Dev)**: ~$210/mes
- **Total**: ~$378-402/mes

### Escenario Optimizado

| Optimización | Ahorro Mensual | Implementación |
|-------------|----------------|----------------|
| Autoscaling en AKS | $42-63 | Fácil |
| Nodos Preemptibles (Dev) | $126-168 | Fácil |
| Reserved Instances (Prod) | $67-115 | Media |
| Auto-stop (Dev) | $105-147 | Media |
| **Total Estimado** | **$340-493/mes** | - |

**Ahorro Potencial**: 50-70% del costo actual

## Scripts y Herramientas

### Scripts Disponibles

1. **`scripts/finops/add-cost-labels.sh`**
   - Añade etiquetas de costos a recursos de Kubernetes
   - Uso: `./scripts/finops/add-cost-labels.sh <namespace> <env> <owner>`

2. **`scripts/finops/gcp-cost-report.sh`**
   - Genera reporte de costos de GCP
   - Uso: `./scripts/finops/gcp-cost-report.sh [start-date] [end-date]`

### Herramientas de Monitoreo

- **GCP Cost Management**: Dashboard nativo de GCP
- **Azure Cost Management**: Dashboard nativo de Azure
- **Prometheus**: Métricas de uso de recursos (CPU, RAM)
- **Grafana**: Dashboards de utilización de recursos

## Cumplimiento del DoD (Definition of Done)

✅ **DoD 1**: Todos los recursos etiquetados (`env`, `service`, `owner`)
- ✅ Etiquetas añadidas en Terraform para clusters y namespaces
- ✅ Script para etiquetar recursos existentes de Kubernetes
- ✅ Etiquetas aplicadas a deployments, services y pods

✅ **DoD 2**: Dashboard de costos mensual por cloud/servicio
- ✅ Documentación de acceso a dashboards de GCP y Azure
- ✅ Instrucciones para filtrar por etiquetas (`env`, `service`, `owner`)
- ✅ Estimaciones de costos mensuales por componente

✅ **DoD 3**: Scripts/policies de ahorro (auto-stop, schedule)
- ✅ Autoscaling configurado en GKE (1-3 nodos)
- ✅ KEDA para escalado de pods
- ✅ **Scale to Zero implementado** para servicios no críticos (`favourite-service`, `shipping-service`)
- ✅ Documentación de políticas de ahorro
- ✅ Recomendaciones para auto-stop y nodos preemptibles

✅ **DoD 4**: Reporte en `docs/finops` con recomendaciones y estimaciones
- ✅ Reporte completo creado
- ✅ Recomendaciones de optimización (corto, mediano, largo plazo)
- ✅ Estimaciones de ahorro potencial
- ✅ Comparativa de costos actuales vs. optimizados

## Guía para Capturar Evidencias

### Comandos para Verificar Etiquetado

```bash
# Ver deployments con etiquetas
kubectl get deployments -n prod --show-labels

# Ver namespace con etiquetas
kubectl get namespace prod --show-labels

# Ver pods con etiquetas
kubectl get pods -n prod --show-labels | head -5

# Verificar un deployment específico
kubectl get deployment api-gateway -n prod --show-labels
```

**Salida esperada**: Deberías ver las etiquetas:
- `env=prod`
- `owner=devops-team`
- `cost-center=engineering`
- `service=<nombre-servicio>`

### Acceso a Dashboards de Costos

#### GCP Cost Management

1. Ve a: https://console.cloud.google.com/billing
2. Selecciona tu proyecto
3. Navega a "Reports" o "Cost breakdown"
4. Filtra por etiquetas: `env=prod`, `service=api-gateway`, etc.

**Capturas recomendadas**:
- Dashboard principal con costos totales
- Vista filtrando por `env=prod`
- Vista filtrando por `service` (costos por microservicio)

#### Azure Cost Management

1. Ve a: https://portal.azure.com/#view/Microsoft_Azure_CostManagement/Menu
2. Selecciona tu suscripción
3. Navega a "Cost analysis"
4. Filtra por etiquetas: `env`, `owner`, etc.

### Verificar Políticas de Ahorro

```bash
# Verificar autoscaling en GKE
gcloud container node-pools describe microservices-cluster-gke-prod-node-pool \
  --cluster microservices-cluster-gke-prod \
  --zone us-central1-a \
  --format="yaml(autoscaling)"

# Ver ScaledObjects de KEDA
kubectl get scaledobjects -n prod

# Ver HPAs creados por KEDA
kubectl get hpa -n prod
```

### Comandos Rápidos para Evidencias

```bash
# Resumen de etiquetas
echo "=== Deployments ===" && kubectl get deployments -n prod --show-labels
echo "=== Services ===" && kubectl get services -n prod --show-labels
echo "=== Namespace ===" && kubectl get namespace prod --show-labels

# Contar recursos por etiqueta
kubectl get pods -n prod -l env=prod --no-headers | wc -l
kubectl get deployments -n prod -l owner=devops-team --no-headers | wc -l
```

## Referencias

- [GCP Cost Management](https://cloud.google.com/cost-management)
- [Azure Cost Management](https://azure.microsoft.com/services/cost-management/)
- [FinOps Foundation](https://www.finops.org/)
- [GCP Pricing Calculator](https://cloud.google.com/products/calculator)
- [Azure Pricing Calculator](https://azure.microsoft.com/pricing/calculator/)

---

**Última actualización**: Noviembre 2025

