# 💰 Comparativa de Costos y Performance - Multi-Cloud

## 📋 Resumen

Este documento presenta una comparativa inicial de costos y performance entre **Azure AKS** y **GCP GKE** para el proyecto de microservicios.

**HU**: HU 12 - Infraestructura Multi-Cloud y Terragrunt (8 SP)

---

## 📊 Tabla Comparativa

| Aspecto | Azure AKS | GCP GKE | Observaciones |
|---------|-----------|---------|---------------|
| **Proveedor** | Microsoft Azure | Google Cloud Platform | Ambos son líderes en el mercado |
| **Región** | `eastus2` (Este de EE.UU. 2) | `us-central1-a` (Iowa) | Regiones con buena latencia |
| **Tipo de Cluster** | Managed Kubernetes | Managed Kubernetes | Ambos completamente gestionados |

### 💻 Configuración de Nodos

| Característica | Azure AKS | GCP GKE |
|----------------|-----------|---------|
| **Node Pool** | `default` | `microservices-cluster-gke-prod-node-pool` |
| **Número de Nodos** | 2 | 2 (autoscaling: 1-3) |
| **VM/Machine Type** | `Standard_D2s_v3` | `e2-medium` |
| **vCPU** | 2 | 2 |
| **RAM** | 8 GB | 4 GB |
| **Disco** | 128 GB (OS disk) | 20 GB (pd-standard) |
| **Auto-scaling** | Manual | Automático (1-3 nodos) |

### 💰 Costos Estimados (por hora)

| Componente | Azure AKS | GCP GKE | Diferencia |
|------------|-----------|---------|------------|
| **Cluster Management** | $0.10/hora | $0.10/hora | Igual |
| **Nodo 1** | ~$0.096/hora | ~$0.067/hora | GKE: -30% |
| **Nodo 2** | ~$0.096/hora | ~$0.067/hora | GKE: -30% |
| **Total Nodos (2)** | ~$0.192/hora | ~$0.134/hora | GKE: -30% |
| **Total Estimado** | **~$0.292/hora** | **~$0.234/hora** | GKE: -20% |

### 💰 Costos Estimados (mensuales)

| Período | Azure AKS | GCP GKE | Diferencia |
|---------|-----------|---------|------------|
| **24/7 (720 horas)** | ~$210/mes | ~$168/mes | GKE: -$42/mes (-20%) |
| **Solo Horario Laboral (160 horas)** | ~$47/mes | ~$37/mes | GKE: -$10/mes (-21%) |

**Nota**: Los costos pueden variar según:
- Uso real de recursos
- Descuentos por compromiso
- Facturación por segundo (GCP) vs por minuto (Azure)
- Costos de red y almacenamiento adicionales

---

## ⚡ Performance

### Latencia de Red

| Métrica | Azure AKS | GCP GKE |
|--------|-----------|---------|
| **Latencia Interna (Pod-to-Pod)** | <1ms | <1ms | Similar |
| **Latencia a Internet** | ~50-100ms | ~50-100ms | Depende de región |
| **Throughput de Red** | Hasta 10 Gbps | Hasta 10 Gbps | Similar |

### Rendimiento de CPU/RAM

| Métrica | Azure AKS | GCP GKE |
|--------|-----------|---------|
| **CPU por Nodo** | 2 vCPU | 2 vCPU | Igual |
| **RAM por Nodo** | 8 GB | 4 GB | **AKS: +100%** |
| **Disco por Nodo** | 128 GB | 20 GB | **AKS: +540%** |
| **I/O Performance** | Alta (SSD Premium) | Media (pd-standard) | AKS superior |

### Escalabilidad

| Característica | Azure AKS | GCP GKE |
|----------------|-----------|---------|
| **Auto-scaling de Nodos** | Manual (puede configurarse) | Automático (1-3) | **GKE: Mejor** |
| **Auto-scaling de Pods (HPA)** | ✅ Soportado | ✅ Soportado | Igual |
| **Límite de Nodos** | 1000+ | 1000+ | Similar |
| **Límite de Pods por Nodo** | 30-110 (depende de VM) | 110 (e2-medium) | Similar |

---

## 🔧 Características Técnicas

### Gestión del Cluster

| Característica | Azure AKS | GCP GKE |
|----------------|-----------|---------|
| **Upgrade Automático** | Manual/Configurable | Automático (Release Channel) | **GKE: Mejor** |
| **Node Auto-repair** | ✅ Soportado | ✅ Habilitado | Igual |
| **Node Auto-upgrade** | ✅ Soportado | ✅ Habilitado | Igual |
| **Maintenance Windows** | Configurable | 03:00 UTC (configurable) | Similar |

### Networking

| Característica | Azure AKS | GCP GKE |
|----------------|-----------|---------|
| **Tipo de Red** | Azure CNI / kubenet | VPC Native | Ambos nativos |
| **Load Balancer** | Azure Load Balancer | GCP Load Balancer | Similar |
| **Ingress Controller** | NGINX / Application Gateway | NGINX / GKE Ingress | Similar |
| **Service Mesh** | Istio / Linkerd | Istio / Anthos | Similar |

### Integración con Servicios Cloud

| Servicio | Azure AKS | GCP GKE |
|----------|-----------|---------|
| **Container Registry** | Azure Container Registry (ACR) | Google Container Registry (GCR) | Ambos integrados |
| **Monitoring** | Azure Monitor | Google Cloud Monitoring | Ambos nativos |
| **Logging** | Azure Log Analytics | Google Cloud Logging | Ambos nativos |
| **Secrets Management** | Azure Key Vault | Google Secret Manager | Ambos disponibles |

---

## 📈 Ventajas y Desventajas

### Azure AKS

**Ventajas**:
- ✅ Mayor RAM por nodo (8 GB vs 4 GB)
- ✅ Mayor disco por nodo (128 GB vs 20 GB)
- ✅ Mejor integración con ecosistema Microsoft
- ✅ Soporte para Windows Containers
- ✅ Azure Active Directory integration

**Desventajas**:
- ❌ Auto-scaling de nodos menos automatizado
- ❌ Costos ligeramente superiores
- ❌ Upgrade del cluster más manual

### GCP GKE

**Ventajas**:
- ✅ Costos más bajos (~20% menos)
- ✅ Auto-scaling de nodos automático y robusto
- ✅ Upgrade automático del cluster (Release Channels)
- ✅ Facturación por segundo (más granular)
- ✅ Mejor para workloads con variabilidad de carga

**Desventajas**:
- ❌ Menor RAM por nodo (4 GB vs 8 GB)
- ❌ Menor disco por nodo (20 GB vs 128 GB)
- ❌ Menos integración con ecosistema Microsoft

---

## 🎯 Recomendaciones

### Para Workloads con Alta Variabilidad
**Recomendación**: **GCP GKE**
- Auto-scaling automático más eficiente
- Facturación por segundo reduce costos
- Mejor para cargas que fluctúan

### Para Workloads Estables con Alto Uso de Memoria
**Recomendación**: **Azure AKS**
- Mayor RAM por nodo (8 GB)
- Mejor para aplicaciones memory-intensive
- Costos predecibles

### Para Desarrollo y Pruebas
**Recomendación**: **GCP GKE**
- Costos más bajos
- Auto-scaling reduce costos cuando no se usa
- Mejor para entornos que se apagan por la noche

### Para Producción con Alta Disponibilidad
**Recomendación**: **Ambos (Multi-Cloud)**
- Redundancia entre proveedores
- Distribución geográfica
- Resiliencia ante fallos de un proveedor

---

## 📊 Métricas de Uso Real (Estimadas)

### Escenario: 10 Microservicios, 2 Réplicas Críticas

| Métrica | Azure AKS | GCP GKE |
|---------|-----------|---------|
| **CPU Utilizado** | ~30-40% | ~40-50% |
| **RAM Utilizado** | ~40-50% | ~60-70% |
| **Disco Utilizado** | ~10-15% | ~30-40% |
| **Nodos Necesarios** | 2 | 2 (puede escalar a 3) |

**Observación**: GKE tiene menor margen de RAM, pero el auto-scaling puede agregar un tercer nodo si es necesario.

---

## 🔄 Estrategia de Optimización de Costos

### 1. Auto-scaling
- **GKE**: Ya configurado (1-3 nodos)
- **AKS**: Configurar Cluster Autoscaler para reducir costos

### 2. Horarios de Uso
- Apagar clusters durante horas no laborales
- Usar nodos preemptibles en GKE (hasta 80% de descuento)

### 3. Reservas y Compromisos
- **Azure**: Reserved Instances (hasta 72% de descuento)
- **GCP**: Committed Use Discounts (hasta 70% de descuento)

### 4. Right-sizing
- Ajustar tamaño de VM según uso real
- Monitorear métricas y optimizar

---

## 📝 Notas Finales

1. **Costos Reales**: Los costos mostrados son estimaciones. Los costos reales dependen del uso, descuentos, y facturación específica.

2. **Performance**: Ambos proveedores ofrecen excelente performance. La elección debe basarse en requisitos específicos del proyecto.

3. **Multi-Cloud**: Mantener ambos clusters permite:
   - Redundancia
   - Comparación continua
   - Flexibilidad para migrar cargas

4. **Monitoreo Continuo**: Es importante monitorear costos y performance regularmente para optimizar.

---

## 📚 Referencias

- [Azure AKS Pricing](https://azure.microsoft.com/pricing/details/kubernetes-service/)
- [GCP GKE Pricing](https://cloud.google.com/kubernetes-engine/pricing)
- [Azure VM Pricing](https://azure.microsoft.com/pricing/details/virtual-machines/)
- [GCP Compute Pricing](https://cloud.google.com/compute/pricing)

---

**Última actualización**: Noviembre 2025

