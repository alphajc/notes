# 日志采集

## filebeat 日志采集

部署参考[官方文档](https://www.elastic.co/guide/en/beats/filebeat/current/running-on-kubernetes.html)

### 创建 pipeline

对应的 ES 需要有 Ingest node 进行 pipeline 处理

```http
PUT /_ingest/pipeline/istio-ingressgateway-pipeline

{
  "istio-ingressgateway-pipeline" : {
    "description" : "istio ingressgateway pipeline",
    "processors" : [
      {
        "kv" : {
          "field" : "request.query",
          "field_split" : "&",
          "value_split" : "=",
          "target_field" : "request.params",
          "strip_brackets" : true
        },
        "remove" : {
          "field" : [
            "request.query",
            "request.url"
          ]
        },
        "date_index_name" : {
          "field" : "request.start_time",
          "index_name_prefix" : "istio-ingressgateway-",
          "date_rounding" : "d"
        }
      }
    ]
  }
}
```

### 创建 filebeat

有部分配置进行过修改，注意命名空间，以及 **elasticsearch** 的访问方式，创建文件`filebeat-kubernetes.yaml`，以下为全部内容：

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: filebeat-config
  namespace: kube-system
  labels:
    k8s-app: filebeat
data:
  filebeat.yml: |-
    # filebeat.inputs:
    # - type: container
    #   paths:
    #     - /var/log/containers/*.log
    #   processors:
    #     - add_kubernetes_metadata:
    #         host: ${NODE_NAME}
    #         matchers:
    #         - logs_path:
    #             logs_path: "/var/log/containers/"

    # To enable hints based autodiscover, remove `filebeat.inputs` configuration and uncomment this:
    filebeat.autodiscover:
     providers:
       - type: kubernetes
         host: ${NODE_NAME}
         hints.enabled: true
         hints.default_config.enabled: false
         hints.default_config:
           type: container
           paths:
             - /var/log/containers/*${data.kubernetes.container.id}.log

    processors:
      - add_cloud_metadata:
      - add_host_metadata:
      - decode_json_fields:
          fields: ["message"]
          target: "request"
      - rename:
          fields:
            - from: "request.path"
              to: "request.url"
          ignore_missing: false
          fail_on_error: true
      - dissect:
          tokenizer: '%{path}?%{query}'
          field: "request.url"
          target_prefix: "request"
      - drop_fields:
          fields: ["message"]
          ignore_missing: false

    cloud.id: ${ELASTIC_CLOUD_ID}
    cloud.auth: ${ELASTIC_CLOUD_AUTH}

    output.elasticsearch:
      hosts: ['${ELASTICSEARCH_HOST:elasticsearch}:${ELASTICSEARCH_PORT:9200}']
      username: ${ELASTICSEARCH_USERNAME}
      password: ${ELASTICSEARCH_PASSWORD}
      # index: "%{[kubernetes.labels.app]}-%{+yyyy.MM.dd}"
      pipeline: "%{[kubernetes.labels.app]}-pipeline"
---
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: filebeat
  namespace: kube-system
  labels:
    k8s-app: filebeat
spec:
  selector:
    matchLabels:
      k8s-app: filebeat
  template:
    metadata:
      labels:
        k8s-app: filebeat
    spec:
      serviceAccountName: filebeat
      terminationGracePeriodSeconds: 30
      hostNetwork: true
      dnsPolicy: ClusterFirstWithHostNet
      containers:
      - name: filebeat
        image: docker.elastic.co/beats/filebeat:7.5.2
        args: [
          "-c", "/etc/filebeat.yml",
          "-e",
        ]
        env:
        - name: ELASTICSEARCH_HOST
          value: elasticsearch
        - name: ELASTICSEARCH_PORT
          value: "9200"
        - name: ELASTICSEARCH_USERNAME
          value: elastic
        - name: ELASTICSEARCH_PASSWORD
          value: changeme
        - name: ELASTIC_CLOUD_ID
          value:
        - name: ELASTIC_CLOUD_AUTH
          value:
        - name: NODE_NAME
          valueFrom:
            fieldRef:
              fieldPath: spec.nodeName
        securityContext:
          runAsUser: 0
          # If using Red Hat OpenShift uncomment this:
          #privileged: true
        resources:
          limits:
            memory: 200Mi
          requests:
            cpu: 100m
            memory: 100Mi
        volumeMounts:
        - name: config
          mountPath: /etc/filebeat.yml
          readOnly: true
          subPath: filebeat.yml
        - name: data
          mountPath: /usr/share/filebeat/data
        - name: varlibdockercontainers
          mountPath: /var/lib/docker/containers
          readOnly: true
        - name: varlog
          mountPath: /var/log
          readOnly: true
      volumes:
      - name: config
        configMap:
          defaultMode: 0600
          name: filebeat-config
      - name: varlibdockercontainers
        hostPath:
          path: /var/lib/docker/containers
      - name: varlog
        hostPath:
          path: /var/log
      # data folder stores a registry of read status for all files, so we don't send everything again on a Filebeat pod restart
      - name: data
        hostPath:
          path: /var/lib/filebeat-data
          type: DirectoryOrCreate
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: filebeat
subjects:
- kind: ServiceAccount
  name: filebeat
  namespace: kube-system
roleRef:
  kind: ClusterRole
  name: filebeat
  apiGroup: rbac.authorization.k8s.io
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: filebeat
  labels:
    k8s-app: filebeat
rules:
- apiGroups: [""] # "" indicates the core API group
  resources:
  - namespaces
  - pods
  verbs:
  - get
  - watch
  - list
---
apiVersion: v1
kind: ServiceAccount
metadata:
  name: filebeat
  namespace: kube-system
  labels:
    k8s-app: filebeat
---
```

创建上述文件后，执行下述命令创建：

```shell
kubectl create -f filebeat-kubernetes.yaml
```
