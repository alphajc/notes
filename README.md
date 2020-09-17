这里将收录一些工作学习中遇到的一些问题及其解决办法，必要时将整理成文章放到自己的[博客](https://mydream.ink/posts/)中。

# python

## pyenv 在国内的使用

以安装 __python 3.6.12__ 为例，`pyenv install`默认使用 python 官方源安装 python，在不使用或者不方便使用梯子时，用起来特别难受，这里对使用已有包进行安装的方法进行记录：

1. 按照[官方文档](https://github.com/pyenv/pyenv)提供的方式安装`pyenv`
2. 从其它地方拷贝或者从国内镜像源下载 python 安装包到任意目录：

    - [淘宝](https://npm.taobao.org/mirrors/python/)

3. 安装:

    ```bash
    PYTHON_BUILD_BUILD_PATH=`pwd` pyenv install 3.6.12
    ```

4. 直接修改 mirror_url 安装：

    ```bash
    PYTHON_BUILD_MIRROR_URL=https://npm.taobao.org/mirrors pyenv install 3.6.12
    ```

# kubernetes

## kustomize 之 patchesJson6902

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

- `target`包含`group`、`kind`、`name`、`namespace` 和`version`字段，并与需要打补丁的 kubernetes 资源一一对应

    > __注：__`group/version`对应 kubernetes 资源中的`apiVersion`

__示例__

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

# 证书

## openssl 操作证书

- 签发证书

    ```bash
    openssl genrsa -out private/demo.key 2048 # 生成私钥
    openssl req -new -key private/demo.key -subj "/CN=demo" -out private/demo.csr # 生成证书请求
    openssl x509 -req -in private/demo.csr -CA ca.crt -CAkey private/ca.key -CAcreateserial -out demo.crt -days 1000 -extensions v3_req # 使用根证书颁发证书
    ```

- 签发带扩展的证书示例

    - 扩展文本内容(`demo.ext`)
    
        ```
        subjectAltName = DNS:k8s1.demo.com,DNS:k8s.demo.com,DNS:kubernetes,DNS:kubernetes.default,DNS:kubernetes.default.svc, DNS:kubernetes.default.svc.cluster.local, IP:10.96.0.1, IP:172.21.0.8, IP:123.206.95.30
        ```

    - 签发：

        ```bash
        openssl x509 -req -in private/demo.csr -CA ca.crt -CAkey private/ca.key -CAcreateserial -out demo.crt -days 1000 -extfile demo.ext
        ```

- 查看证书内容

    ```bash
    openssl x509 -in demo.crt -noout -text
    ```

- 自签证书根证书

    ```bash
    openssl genrsa -aes256 -out private/ca.key 1024
    openssl req -new -key private/ca.key -out private/ca.csr -subj \
"/C=CN/ST=myprovince/L=mycity/O=myorganization/OU=mygroup/CN=myname"
    ```

# 常用命令

- 为非默认端口的站点生成 known_hosts

    ```bash
    ssh-keyscan -p 9522 gitlab.xxx.com
    ```

