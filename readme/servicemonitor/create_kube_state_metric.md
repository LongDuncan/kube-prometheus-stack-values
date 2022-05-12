# 建立 Kube State Metric Service Monitor

## 步驟
1. 確定kube-state-metrics pod/service是否有建立
    ```bash
    kubectl get pod -A |grep kube-state-metrics

    &

    kubectl get service -A |grep kube-state-metrics
    ```

2. 取得kube-state-metrics service 相關資訊含labels
    kube-state-metrics service資訊範例如下
    > 1. namespace: kube-state
    > 2. servicename: kubestate-kube-state-metrics
    > 3. labels: kubernetes.io/name=kube-state-metrics, release=kubestate
    ---
    ```bash
    kubectl get service kubestate-kube-state-metrics -n kube-state --show-labels
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
        app.kubernetes.io/component: metrics
        app.kubernetes.io/instance: prometheus
        app.kubernetes.io/name: kube-state-metrics
        app.kubernetes.io/part-of: kube-state-metrics
        release: prometheus             # 透過helm chart部署kube-prometheus-stack對應的helm chart name
    name: prometheus-kube-state-metrics # servicemonitor 名稱
    namespace: monitoring               # servicemonitor 建立在哪個namespaces底下
    spec:
    endpoints:
    - honorLabels: true
        port: http
    jobLabel: app.kubernetes.io/name
    namespaceSelector:
        matchNames:
        - kube-state                               # kubestate-kube-state-metrics service所在的namespace
    selector:
        matchLabels:
        app.kubernetes.io/name: kube-state-metrics # kubestate-kube-state-metrics service label
        release: kubestate                         # kubestate-kube-state-metrics service label
    ```
4. 建立servicemonitor服務
    ```bash
    kubectl apply -f servicemonitor_kube-state-metrics.yaml
    ```
5. 透過prometheus UI確認Service Discovery與Targets是否正常