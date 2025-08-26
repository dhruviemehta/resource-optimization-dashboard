# 🎯 Kubernetes Resource Optimization Dashboard

A comprehensive Grafana dashboard designed to identify oversized and undersized Kubernetes deployments, enabling data-driven cost optimization decisions for infrastructure teams and business stakeholders.

## 📋 Table of Contents

- [Overview](#overview)
- [Features](#features)
- [Prerequisites](#prerequisites)
- [Installation](#installation)
- [Configuration](#configuration)
- [Usage Guide](#usage-guide)
- [Methodology](#methodology)
- [Troubleshooting](#troubleshooting)
- [Cost Optimization Workflow](#cost-optimization-workflow)
- [Limitations](#limitations)
- [Contributing](#contributing)

## 🔍 Overview

This dashboard helps organizations optimize their Kubernetes resource allocation by:

- **Identifying oversized deployments** that waste money by requesting more resources than needed
- **Detecting undersized deployments** that may suffer performance issues due to resource constraints
- **Calculating potential cost savings** from right-sizing resources
- **Providing actionable insights** through intuitive visualizations for both technical and non-technical users

### Key Metrics
- **Oversized**: Deployments using < 20% of requested CPU/memory
- **Undersized**: Deployments using > 80% of requested CPU/memory  
- **Optimal**: Deployments with 20-80% resource utilization

## ✨ Features

### 📊 Visual Components

1. **Executive Summary**
   - Pie charts showing resource utilization distribution
   - Instant overview of optimization opportunities

2. **Detailed Analysis Table**
   - Deployment-level resource usage and recommendations
   - Color-coded status indicators for quick decision making
   - Potential cost savings calculations

3. **Trend Analysis**
   - Time-series charts showing resource usage patterns
   - Historical data to validate optimization decisions

4. **Cost Impact Summary**
   - Monthly savings potential from oversized deployments
   - Count of deployments requiring attention

### 🎨 User Experience Features

- **Non-technical friendly**: Emoji icons and clear status indicators
- **Color-coded backgrounds**: Green (optimal), Orange (oversized), Red (undersized)
- **Sortable tables**: Prioritized by potential cost savings
- **Real-time updates**: 30-second refresh interval

## 🔧 Prerequisites

### Required Components

- **Kubernetes cluster** (v1.16+)
- **Prometheus** (v2.20+) with proper configuration
- **Grafana** (v7.0+)
- **kube-state-metrics** (v2.0+)
- **cAdvisor** (usually bundled with kubelet)

### Required Metrics

The dashboard requires these Prometheus metrics to be available:

```promql
# Container resource usage
container_cpu_usage_seconds_total
container_memory_working_set_bytes

# Resource requests
kube_pod_container_resource_requests

# Pod metadata
kube_pod_info
```

### Kubernetes Permissions

Ensure your monitoring setup can access:
- Pod metrics and metadata
- ReplicaSet information
- Resource requests and limits

## 🚀 Installation

### Option 1: Complete Monitoring Stack (Recommended)

Install the full monitoring stack using Helm:

```bash
# Add Prometheus community Helm repository
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update

# Install kube-prometheus-stack (includes Prometheus, Grafana, exporters)
helm install monitoring prometheus-community/kube-prometheus-stack \
  --namespace monitoring \
  --create-namespace \
  --set grafana.adminPassword=admin123
```

### Option 2: Existing Grafana Instance

If you already have Grafana running:

1. **Access Grafana UI**
   ```bash
   # Port-forward to your Grafana service
   kubectl port-forward svc/your-grafana-service 3000:3000
   ```

2. **Import Dashboard**
   - Navigate to **+ → Import**
   - Copy and paste the JSON from `resource-optimization-dashboard.json`
   - Configure Prometheus datasource
   - Save dashboard

### Option 3: Grafana Cloud

For managed Grafana installations:
1. Upload the dashboard JSON to Grafana Cloud
2. Configure your Prometheus datasource
3. Ensure your cluster metrics are being scraped

## ⚙️ Configuration

### Prometheus Configuration

Ensure your Prometheus configuration includes these scrape configs:

```yaml
# prometheus.yml
scrape_configs:
  - job_name: 'kubernetes-pods'
    kubernetes_sd_configs:
      - role: pod
    relabel_configs:
      - source_labels: [__meta_kubernetes_pod_annotation_prometheus_io_scrape]
        action: keep
        regex: true

  - job_name: 'kube-state-metrics'
    kubernetes_sd_configs:
      - role: endpoints
    relabel_configs:
      - source_labels: [__meta_kubernetes_service_name]
        action: keep
        regex: kube-state-metrics

  - job_name: 'kubelet'
    scheme: https
    tls_config:
      ca_file: /var/run/secrets/kubernetes.io/serviceaccount/ca.crt
      insecure_skip_verify: true
    bearer_token_file: /var/run/secrets/kubernetes.io/serviceaccount/token
    kubernetes_sd_configs:
      - role: node
    relabel_configs:
      - action: labelmap
        regex: __meta_kubernetes_node_label_(.+)
    metric_relabel_configs:
      - source_labels: [__name__]
        regex: 'container_cpu_usage_seconds_total|container_memory_working_set_bytes'
        action: keep
```

### Cost Pricing Configuration

The dashboard uses default cloud pricing estimates. To customize:

1. **CPU Pricing** (default: $0.024 per CPU-hour):
   ```promql
   # Find this in the dashboard queries and modify:
   kube_pod_container_resource_requests{resource="cpu"} * 1000 * 0.024
   # Change 0.024 to your actual CPU cost per core-hour
   ```

2. **Memory Pricing** (default: $0.012 per GB-hour):
   ```promql
   # Modify this multiplier:
   kube_pod_container_resource_requests{resource="memory"} / 1024 / 1024 / 1024 * 0.012
   # Change 0.012 to your actual memory cost per GB-hour
   ```

### Resource Thresholds

To modify the utilization thresholds:

1. **Oversized threshold** (default: < 20%):
   ```promql
   # Change 20 to your preferred percentage
   ) < 20
   ```

2. **Undersized threshold** (default: > 80%):
   ```promql  
   # Change 80 to your preferred percentage
   ) > 80
   ```

## 📖 Usage Guide

### For Business Stakeholders

1. **Quick Assessment**
   - Look at the pie charts at the top for immediate overview
   - Focus on orange (oversized) sections for cost-saving opportunities
   - Red sections indicate potential performance risks

2. **Priority Actions**
   - Review the analysis table sorted by "Potential Monthly Savings"
   - Focus on deployments with highest dollar impact first
   - Use the emoji indicators for quick status understanding

3. **Decision Making**
   - ✅ Optimal: No action needed
   - ⚠️ Oversized: Safe to reduce resource requests
   - 🚨 Undersized: Needs more resources or investigation

### For Technical Teams

1. **Deep Analysis**
   - Use the detailed table to see exact utilization percentages
   - Review trend charts to understand usage patterns over time
   - Cross-reference with application performance metrics

2. **Implementation Planning**
   - Start with highest-impact oversized deployments
   - Make gradual adjustments (10-20% reductions)
   - Monitor for 1-2 weeks before further optimization

3. **Validation Process**
   - Use time-series charts to confirm usage patterns
   - Check if low utilization is due to recent deployment or genuine over-provisioning
   - Consider business requirements and SLA needs

### Dashboard Panels Explained

| Panel | Purpose | Action Items |
|-------|---------|-------------|
| 🎯 CPU/Memory Overview | Executive summary of resource distribution | Identify overall optimization opportunity |
| 💰 Deployment Analysis Table | Detailed per-deployment metrics | Prioritize optimization efforts |
| 📈 Usage Trends | Historical usage patterns | Validate optimization decisions |
| 💸 Cost Savings Summary | Financial impact metrics | Report ROI to stakeholders |
| ⚠️ Alert Counts | Quick status indicators | Monitor optimization progress |

## 🔬 Methodology

### Resource Utilization Calculation

**CPU Utilization:**
```promql
(rate(container_cpu_usage_seconds_total[5m]) * 100) / 
(kube_pod_container_resource_requests{resource="cpu"} * 1000)
```

**Memory Utilization:**
```promql
(container_memory_working_set_bytes) / 
(kube_pod_container_resource_requests{resource="memory"})
```

### Deployment Filtering

The dashboard specifically targets Kubernetes Deployments by:
- Filtering for pods created by ReplicaSets (`created_by_kind="ReplicaSet"`)
- Extracting deployment names from ReplicaSet naming convention
- Excluding StatefulSets, DaemonSets, Jobs, and CronJobs

### Cost Calculation Model

**Monthly Cost Estimate:**
- CPU: `CPU_cores × hours_per_month × cost_per_core_hour`
- Memory: `Memory_GB × hours_per_month × cost_per_GB_hour`
- Hours per month: 720 (24 × 30)

**Savings Calculation:**
- Identifies resources above/below optimal thresholds
- Calculates potential reduction for oversized resources
- Estimates monthly savings based on resource pricing

## 🔧 Troubleshooting

### Common Issues

#### 1. No Data in Dashboard

**Symptoms:**
- All panels show "No data"
- Queries return empty results

**Solutions:**
```bash
# Check Prometheus connectivity
kubectl port-forward -n monitoring svc/prometheus-server 9090:80

# Verify metrics are being scraped
curl 'http://localhost:9090/api/v1/query?query=up{job="kubelet"}'

# Check kube-state-metrics
kubectl get pods -n monitoring | grep kube-state-metrics
kubectl logs -n monitoring deployment/kube-state-metrics
```

#### 2. Deployments Not Appearing

**Symptoms:**
- Expected deployments missing from table
- Lower counts than expected

**Diagnosis:**
```bash
# Check if pods have resource requests
kubectl describe deployment <deployment-name> | grep -A 10 "requests"

# Verify ReplicaSet labeling
kubectl get replicasets -o custom-columns="NAME:.metadata.name,OWNER:.metadata.ownerReferences[0].name"

# Check pod creation method
kubectl get pods -o yaml | grep "created_by_kind"
```

**Solutions:**
- Ensure deployments have resource requests defined
- Verify ReplicaSet naming follows standard convention
- Check if workloads are actually Deployments (not StatefulSets, etc.)

#### 3. Incorrect Cost Calculations

**Symptoms:**
- Unrealistic savings amounts
- Negative values in cost fields

**Solutions:**
```promql
# Verify pricing constants
# CPU pricing (modify 0.024 as needed)
kube_pod_container_resource_requests{resource="cpu"} * 1000 * 0.024

# Memory pricing (modify 0.012 as needed)  
kube_pod_container_resource_requests{resource="memory"} / 1024 / 1024 / 1024 * 0.012
```

#### 4. Performance Issues

**Symptoms:**
- Dashboard loads slowly
- Grafana becomes unresponsive

**Solutions:**
- Increase query intervals from 30s to 1m or 5m
- Add namespace filters to reduce query scope
- Implement Prometheus recording rules for complex calculations
- Consider using separate Grafana instance for resource monitoring

### Debugging Queries

**Test individual components:**

```promql
# Basic container metrics
container_cpu_usage_seconds_total{container!="POD",container!=""}

# Resource requests
kube_pod_container_resource_requests{resource="cpu"}

# Pod-to-deployment mapping
kube_pod_info{created_by_kind="ReplicaSet"}

# Complete utilization calculation
(rate(container_cpu_usage_seconds_total{container!="POD",container!=""}[5m]) * 100) /
(kube_pod_container_resource_requests{resource="cpu"} * 1000)
```

### Data Collection Requirements

**Minimum collection period:** 2-4 weeks for accurate patterns
**Recommended refresh interval:** 30 seconds to 5 minutes
**Historical data retention:** 30+ days for trend analysis

## 💡 Cost Optimization Workflow

### Phase 1: Assessment (Week 1-2)

1. **Install and configure** the dashboard
2. **Collect baseline data** for 2 weeks minimum
3. **Identify patterns** in resource utilization
4. **Document findings** and create optimization plan

### Phase 2: Planning (Week 3)

1. **Prioritize deployments** by potential savings
2. **Assess business impact** of each optimization
3. **Create rollback plans** for critical applications
4. **Schedule optimization windows** during low-traffic periods

### Phase 3: Implementation (Week 4+)

1. **Start with highest-impact, lowest-risk** deployments
2. **Make incremental changes** (10-20% adjustments)
3. **Monitor application performance** closely
4. **Wait 3-7 days** between optimization rounds

### Phase 4: Validation (Ongoing)

1. **Track cost savings** using dashboard metrics
2. **Monitor application health** and performance
3. **Document lessons learned** for future optimizations
4. **Repeat cycle** quarterly or as needed

### Best Practices

- **Never optimize during peak business hours**
- **Always have rollback procedures ready**
- **Coordinate with application owners**
- **Monitor for at least 48 hours after changes**
- **Document all changes for compliance**

## ⚠️ Limitations

### Technical Limitations

1. **Workload Patterns**
   - May not account for seasonal or cyclical usage patterns
   - Short-term spikes might not be captured in 5-minute averages
   - Cold start effects can skew new deployment metrics

2. **Kubernetes Scope**
   - Only covers Deployment workloads (excludes StatefulSets, DaemonSets, Jobs)
   - Requires resource requests to be defined
   - Multi-container pods are aggregated, potentially masking individual issues

3. **Cost Accuracy**
   - Uses estimated cloud pricing, not actual billing
   - Doesn't account for reserved instances or volume discounts
   - No consideration for networking, storage, or other costs

4. **Metric Limitations**
   - Depends on Prometheus data retention policies
   - 5-minute rate windows may miss very brief spikes
   - Network partitions can create data gaps

### Business Limitations

1. **Context Awareness**
   - Cannot determine business criticality of applications
   - No awareness of SLA requirements or compliance needs
   - May suggest optimizations that conflict with disaster recovery plans

2. **Performance Correlation**
   - Doesn't directly measure application performance impact
   - Can't predict performance degradation from resource reductions
   - No integration with APM or user experience metrics

### Operational Considerations

1. **Change Management**
   - Requires coordination with application teams
   - May conflict with existing capacity planning processes
   - Needs integration with deployment and change approval workflows

2. **Monitoring Dependencies**
   - Requires robust Prometheus setup with proper retention
   - Depends on accurate kube-state-metrics configuration
   - Dashboard performance scales with cluster size

## 🤝 Contributing

### Reporting Issues

When reporting issues, please include:

- Kubernetes version and distribution
- Prometheus and Grafana versions
- Dashboard JSON version
- Error messages or unexpected behavior
- Steps to reproduce the issue

### Enhancement Requests

We welcome suggestions for:
- Additional metrics and calculations
- New visualization types
- Integration with other monitoring tools
- Cost model improvements

### Development Setup

1. **Fork the repository**
2. **Set up test environment** with minikube or kind
3. **Install monitoring stack** using provided instructions
4. **Test changes** against multiple deployment patterns
5. **Submit pull request** with detailed description

### Code Standards

- Use clear, descriptive variable names in PromQL queries
- Include comments for complex calculations
- Test against multiple Kubernetes environments
- Follow Grafana dashboard best practices
- Maintain backward compatibility when possible

---

## 📄 License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

## 📞 Support

- **Documentation**: Check this README and inline comments
- **Issues**: Use GitHub Issues for bug reports and feature requests
- **Community**: Join our Slack channel for discussions and support
- **Enterprise**: Contact us for enterprise support and custom implementations

---

**Made with ❤️ for the Kubernetes community**

*Help us improve this dashboard by sharing your feedback and optimization success stories!*