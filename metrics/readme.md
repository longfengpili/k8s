# yml
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: kube-state-metrics
  namespace: kube-system
spec:
  replicas: 1
  selector:
    matchLabels:
      app: kube-state-metrics
  template:
    metadata:
      labels:
        app: kube-state-metrics
    spec:
      serviceAccountName: kube-state-metrics
      containers:
      - name: kube-state-metrics
        image: bitnami/kube-state-metrics:2.5.0
        ports:
        - containerPort: 8080

#创建账号，授权该账号有权限访问k8s集群
---
apiVersion: v1
kind: ServiceAccount
metadata:
  name: kube-state-metrics
  namespace: kube-system
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: kube-state-metrics
rules:
- apiGroups: [""]
  resources: ["nodes", "pods", "services", "resourcequotas", "replicationcontrollers", "limitranges", "persistentvolumeclaims", "persistentvolumes", "namespaces", "endpoints"]
  verbs: ["list", "watch"]
- apiGroups: ["extensions"]
  resources: ["daemonsets", "deployments", "replicasets"]
  verbs: ["list", "watch"]
- apiGroups: ["apps"]
  resources: ["statefulsets"]
  verbs: ["list", "watch"]
- apiGroups: ["batch"]
  resources: ["cronjobs", "jobs"]
  verbs: ["list", "watch"]
- apiGroups: ["autoscaling"]
  resources: ["horizontalpodautoscalers"]
  verbs: ["list", "watch"]
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: kube-state-metrics
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: kube-state-metrics
subjects:
- kind: ServiceAccount
  name: kube-state-metrics
  namespace: kube-system

#创建service，暴露端口，用于在k8s外部访问
---
apiVersion: v1
kind: Service
metadata:
  annotations:
    prometheus.io/scrape: 'true'
  name: kube-state-metrics
  namespace: kube-system
  labels:
    app: kube-state-metrics
spec:
  type: NodePort
  ports:
  - name: kube-state-metrics
    port: 8080
    targetPort: 8080
    nodePort: 31666
    protocol: TCP
  selector:
    app: kube-state-metrics
```


# 部署监控
```bash
kubectl apply -f k8s_state_metrice.yml
```

# 访问链接
http://localhost:31666

# Prometheus配置job采集数据
```bash
vim /apps/prometheus/prometheus.yml
```

```yaml
  - job_name: "k8s"
    static_configs:
      - targets: 
          - '192.168.2.151:31666'
        labels: 
          env: sandbox

    relabel_configs:
      - source_labels: [__address__]
        target_label: instance
        replacement: 'k8s(151)'
        action: replace
        regex: '192.168.2.151:31666'  # 仅在匹配时重命名

```

# grafana
https://grafana.com/grafana/dashboards/13105-k8s-dashboard-cn-20240513-starsl-cn/

