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

