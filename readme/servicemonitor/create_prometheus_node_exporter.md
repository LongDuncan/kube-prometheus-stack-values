# 建立 Prometheus Node Exporter Service Monitor

## 步驟
1. 確定prometheus-node-exporter pod/service是否有建立
    ```bash
    kubectl get pod -A |grep prometheus-node-exporter

    &

    kubectl get service -A |grep prometheus-node-exporter
    ```

2. 取得prometheus-node-exporter service 相關資訊含labels
    prometheus-node-exporter service資訊範例如下
    > 1. namespace: kube-state
    > 2. servicename: kubestate-prometheus-node-exporter
    > 3. labels: app=prometheus-node-exporter, release=kubestate, jobLabel=node-exporter
    ---
    ```bash
    kubectl get service kubestate-prometheus-node-exporter -n kube-state --show-labels
    ```

3. 編輯servicemonitor yaml檔
    注意的地方
    > 1. release: {helm-chart-deployname}
    > 2. name: {helm-chart-deployname}-kube-state-metrics
    ---
    ```yaml
    apiVersion: monitoring.coreos.com/v1
    kind: ServiceMonitor
    metadata:
    labels:
        app: prometheus-node-exporter
        jobLabel: node-exporter
        release: prometheus                   # 透過helm chart部署prometheus-node-exporter對應的helm chart name
    name: prometheus-prometheus-node-exporter # servicemonitor 名稱
    namespace: monitoring                     # servicemonitor 建立在哪個namespaces底下
    spec:
    endpoints:
    - port: http-metrics
        scheme: http
        relabelings:
        - sourceLabels: [__meta_kubernetes_pod_node_name]
            separator: ;
            regex: ^(.*)$
            targetLabel: nodename
            replacement: $1
            action: replace
    jobLabel: jobLabel
    namespaceSelector:
        matchNames:
        - kube-state                          # kubestate-prometheus-node-exporter service所在的namespace
    selector:
        matchLabels:
        app: prometheus-node-exporter         # kubestate-prometheus-node-exporter service label
        release: kubestate                    # kubestate-prometheus-node-exporter service label
    ```
4. 建立servicemonitor服務
    ```bash
    kubectl apply -f servicemonitor_prometheus-node-exporter.yaml
    ```
5. 透過prometheus UI確認Service Discovery與Targets是否正常