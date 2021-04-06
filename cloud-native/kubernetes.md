# Kubernetes

## 观测

### 资源使用情况

#### 查看 Pod 资源分配情况

```bash
kubectl get pod -o jsonpath='{range .items[*]}{"\n"}{.metadata.name}{":\t"}{range .spec.containers[*]}{.resources}{", "}{end}{end}' --all-namespaces
```

#### 查看 Pod 资源使用情况

```bash
kubectl top po --all-namespaces | sed '1d' | awk '{print "\nnamespace:"$1"\npod:"$2"\ncpu:"$3"\nmemory:"$4;system("kubectl -n "$1" get po "$2" -o=jsonpath=\"{range .spec.containers[*]}{.resources.requests}{end}\""); print ""}' > resources_usage.txt
```

## 部署

### 使用 Kustomize

#### patchesJson6902

kustomize 中的`patchesJson6902`，最简单直接代码量最少的给 kubernetes 资源打补丁的方式

该字段是一个列表，每项含`target`和`path`两个字段

- `path`指向一个 patch 文件，示例文件如下：

    ```yaml
     - op: add
       path: /some/new/path
       value: value
     - op: replace
       path: /some/existing/path
       value: new value
    ```

- `target`包含`group`、`kind`、`name`、`namespace` 和`version`字段，并与需要打补丁的 Kubernetes 资源一一对应

    > **注：**`group/version`对应 kubernetes 资源中的`apiVersion`

#### 示例

```yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
patchesJson6902:
- target:
    version: v1
    kind: Deployment
    name: my-deployment
  path: add_init_container.yaml
- target:
    version: v1
    kind: Service
    name: my-service
  path: add_service_annotation.yaml
```

## 管理

### 权限

#### 按 serviceaccount 的方式给镜像拉取权限

```bash
kubectl create secret docker-registry <name> --docker-server=DOCKER_REGISTRY_SERVER --docker-username=DOCKER_USER --docker-password=DOCKER_PASSWORD --docker-email=DOCKER_EMAIL # 创建 imagePullSecret
kubectl patch serviceaccount default -p '{"imagePullSecrets": [{"name": "docker-registry"}]}' # 为 SA 添加镜像拉取权限
```
