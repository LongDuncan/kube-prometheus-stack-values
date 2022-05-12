# Service Monitor 配置說明
```yaml
# Prometheus Operator CRD 版本
apiVersion: monitoring.coreos.com/v1
# 对应 K8S 的资源类型，这里面 Service Monitor
kind: ServiceMonitor
# 对应 K8S 的 Metadata，这里只用关心 name，如果没有指定 jobLabel，对应抓取指标 label 中 job 的值为 Service 的名称。
metadata:
  name: redis-exporter # 填写一个唯一名称
  namespace: cm-prometheus  # namespace固定，不需要修改
# 描述抓取目标 Pod 的选取及抓取任务的配置
spec:
  # 填写对应 Pod 的 label(metadata/labels)，service monitor 会取对应的值作为 job label 的值
  [ jobLabel: string ]
  # 把对应 service 上的 Label 添加到 Target 的 Label 中
  [ targetLabels: []string ]
  # 把对应 Pod 上的 Label 添加到 Target 的 Label 中
  [ podTargetLabels: []string ]
  # 一次抓取数据点限制，0：不作限制，默认为 0
  [ sampleLimit: uint64 ]
  # 一次抓取 Target 限制，0：不作限制，默认为 0
  [ targetLimit: uint64 ]
  # 配置需要抓取暴露的 Prometheus HTTP 接口，可以配置多个 Endpoint
  endpoints:
  [ - <endpoint_config> ... ] # 详见下面 endpoint 说明
  # 选择要监控 Pod 所在的 namespace，不填为选取所有 namespace
  [ namespaceSelector: ]  
    # 是否选取所有 namespace
    [ any: bool ]
    # 需要选取 namespace 列表
    [ matchNames: []string ]
  # 填写要监控 Pod 的 Label 值，以定位目标 Pod  [K8S metav1.LabelSelector](https://v1-17.docs.kubernetes.io/docs/reference/generated/kubernetes-api/v1.17/#labelselector-v1-meta)
  selector:  
    [ matchExpressions: array ]
      [ example: - {key: tier, operator: In, values: [cache]} ]
    [ matchLabels: object ]
      [ example: k8s-app: redis-exporter ]
```
---

# Service Monitor 範例1
```yaml
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  name: go-demo    # 填写一个唯一名称
  namespace: cm-prometheus  # namespace固定，不要修改
spec:
  endpoints:
  - interval: 30s
    # 填写service yaml中Prometheus Exporter对应的Port的Name
    port: 8080-8080-tcp
    # 填写Prometheus Exporter对应的Path的值，不填默认/metrics
    path: /metrics
    relabelings:
    # ** 必须要有一个 label 为 application，这里假设 k8s 有一个 label 为 app，
    # 我们通过 relabel 的 replace 动作把它替换成了 application
    - action: replace
      sourceLabels:  [__meta_kubernetes_pod_label_app]
      targetLabel: application
  # 选择要监控service所在的namespace
  namespaceSelector:
    matchNames:
    - golang-demo
  # 填写要监控service的Label值，以定位目标service
  selector:
    matchLabels:
      app: golang-app-demo
```

# Service Monitor 範例2
```yaml
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  name: nginx-ingress-controller-metrics
  labels:
    app: nginx-ingress
    release: prometheus
  namespace: monitoring
spec:
  endpoints:
  - interval: 30s
    port: metrics
    relabelings:
    - action: replace
      sourceLabels:  [__meta_kubernetes_pod_node_name]
      targetLabel: nodename
  selector:
    matchLabels:
      # app: nginx-ingress
      app.kubernetes.io/name: ingress-nginx
  namespaceSelector:
    matchNames:
    - ingress-basic
```