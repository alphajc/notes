这里将收录一些工作学习中遇到的一些问题及其解决办法，必要时将整理成文章放到自己的[博客](https://mydream.ink/posts/)中。

---

# 云原生

## Kubernetes

### 观测

#### 资源使用情况

- 查看 Pod 资源分配情况

    ```bash
    kubectl get pod -o jsonpath='{range .items[*]}{"\n"}{.metadata.name}{":\t"}{range .spec.containers[*]}{.resources}{", "}{end}{end}' --all-namespaces
    ```

- 查看 Pod 资源使用情况

    ```bash
    kubectl top po --all-namespaces | sed '1d' | awk '{print "\nnamespace:"$1"\npod:"$2"\ncpu:"$3"\nmemory:"$4;system("kubectl -n "$1" get po "$2" -o=jsonpath=\"{range .spec.containers[*]}{.resources.requests}{end}\""); print ""}' > resources_usage.txt
    ```

### 部署

#### 使用 Kustomize 

__patchesJson6902__

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

### 管理

#### 权限

- 按 serviceaccount 的方式给镜像拉取权限

    ```bash
    kubectl create secret docker-registry <name> --docker-server=DOCKER_REGISTRY_SERVER --docker-username=DOCKER_USER --docker-password=DOCKER_PASSWORD --docker-email=DOCKER_EMAIL # 创建 imagePullSecret
    kubectl patch serviceaccount default -p '{"imagePullSecrets": [{"name": "docker-registry"}]}' # 为 SA 添加镜像拉取权限
    ```

---



## Argo

### Argocd

- 重置 argocd 密码

    ```bash
    kubectl -n argocd patch secret argocd-secret  -p '{"data": {"admin.password": null, "admin.passwordMtime": null}}' # 清除密码
    kubectl -n argocd delete pod -l app.kubernetes.io/name=argocd-server # 重建 pod
    ```

    重置后的密码为新的 pod 的名字

    ```bash
    kubectl get pods -n argocd -l app.kubernetes.io/name=argocd-server -o name | cut -d'/' -f 2
    ```

# 安全

## 证书

### Openssl 操作证书

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

---

# 数据库

## Redis

- 查看每客户端连接数

    ```bash
    redis-cli client list | awk -F '[=:]' '{sum[$2] += 1} END {for(i in sum) print i, sum[i];}' | sort -n -r -k 2
    ```

## Mysql

- 刷 binlog 日志，并新启 binlog 文件

    ```sql
    FLUSH LOGS
    ```

---

# 中间件

## Zookeeper

- 生成ACL

    ```bash
    echo -n <user>:<password> | openssl dgst -binary -sha1 | openssl base64
    ```

---

# 编程语言

## Python

### 开发环境

__pyenv 在国内的使用__

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

---

# 常用命令

- 为非默认端口的站点生成 known_hosts

    ```bash
    ssh-keyscan -p 9522 gitlab.xxx.com
    ```

---

# 书签

<!-- tabs:start -->

### ** 云原生 **

- [Google 镜像仓库](https://console.cloud.google.com/gcr/images/google-containers/GLOBAL)
- [Alpine Linux packages](https://pkgs.alpinelinux.org/contents)：根据文件名检索 alpine 的包，用于制作最小 docker 镜像
- [Elastic Cloud on Kubernetes](https://www.elastic.co/cn/elastic-cloud-kubernetes)：在 Kubernetes 上部署和编排 Elasticsearch
- [Running Elasticsearch on ECK](https://www.elastic.co/guide/en/cloud-on-k8s/current/k8s-elasticsearch-specification.html)：使用 ECK 配置 elastic
- [华为云在 K8S 大规模场景下的 Service 性能优化实践 - 知乎](https://zhuanlan.zhihu.com/p/37230013)：`ipvs`与`iptables`型的 Service 间的性能差异
- [kubectl 使用指南](https://kubectl.docs.kubernetes.io/)
- [CNCF x Alibaba 云原生技术公开课 - 云原生教程 - 阿里云大学](https://edu.aliyun.com/roadmap/cloudnative)

### ** 计算机网络 **

- [TCP 拥塞控制](https://zhuanlan.zhihu.com/p/37379780)
- [SDN 网络指南](https://www.bookstack.cn/read/sdn-handbook/SUMMARY.md)
- [HTTP2 简介](https://developers.google.com/web/fundamentals/performance/http2/?hl=zh-cn)

### ** DevOps **

- [The Kubernetes executor | GitLab](https://docs.gitlab.com/runner/executors/kubernetes.html)：基于 Kubernetes 的 gitlab-runner 的使用
- [实例化DevOps原则 - ThoughtWorks洞见](https://insights.thoughtworks.cn/instantiate-the-principles-of-devops/)
- [DevOps团队交付了什么？ - 知乎](https://zhuanlan.zhihu.com/p/28716423)
- [敏捷与DevOps整合之道](https://www.tapd.cn/forum/view/52101)

### ** 运维 **

- [USB转串口Mac驱动](http://www.prolific.com.tw/US/ShowProduct.aspx?p_id=229&pcid=41)：Mac 连接服务器串口
- [Shell高级编程学习笔记(基础篇) - 90Zeng - 博客园](https://www.cnblogs.com/90zeng/p/shellNotes.html)
- [Linux工具快速教程](https://linuxtools-rst.readthedocs.io/zh_CN/latest/index.html)

__LDAP__

- [Centralized authentication using OpenLDAP - Gentoo Wiki](https://wiki.gentoo.org/wiki/Centralized_authentication_using_OpenLDAP)
- [LDAP authentication - ArchWiki](https://wiki.archlinux.org/index.php/LDAP_authentication)

__邮件服务__

- [Linux系统下邮件服务器的搭建（Postfix+Dovecot） - Lomu](http://lomu.me/post/linux-email-server)
- [How to secure Postfix using Let's Encrypt - UpCloud](https://upcloud.com/community/tutorials/secure-postfix-using-lets-encrypt/)：配置加密邮件服务

### ** 工具 **

- [github emoji 表情列表](https://gist.github.com/rxaviers/7360908)
- [Text to ASCII Art Generator (TAAG)](http://patorjk.com/software/taag/#p=display&f=Graffiti&t=Type%20Something%20)
- [Office for Mac](https://docs.microsoft.com/en-us/officeupdates/update-history-office-for-mac)
- [CSV to Markdown Table Generator — Donat Studios](https://donatstudios.com/CsvToMarkdownTable)：CSV 转 Markdown Table
- [Oh-My-Zsh OSX 插件](https://github.com/ohmyzsh/ohmyzsh/tree/master/plugins/osx)
- [Base64 Image Encoder](https://www.base64-image.de/)：为图片生成 base64
- [正则表达式 - 维基百科，自由的百科全书](https://zh.wikipedia.org/wiki/%E6%AD%A3%E5%88%99%E8%A1%A8%E8%BE%BE%E5%BC%8F#PCRE%E8%A1%A8%E8%BE%BE%E5%BC%8F%E5%85%A8%E9%9B%86)
- [Microsoft Office 2019 Preview Download & Install - Techhelpday](https://techhelpday.com/microsoft-office-2019/)
- [V2ray 订阅地址](https://www.butnono.com/latest-2020-freevpn-v2ray-ss-ssr-address.html)：翻墙必备

### ** 软件架构 **

- [Raft 协议](http://thesecretlivesofdata.com/raft/)
- [社交产品后端架构设计-CSDN.NET](https://www.csdn.net/article/2015-08-10/2825423)

### ** 开发 **

- [Git Commit Log的小型团队最佳实践 - 汤姆C - SegmentFault 思否](https://segmentfault.com/a/1190000015434246)
- [Programiz: Learn to Code for Free](https://www.programiz.com/)：学写代码
- [RESTful HTTP Request and Response Examples](https://docs.tibco.com/pub/tpm-rest/1.0.0/doc/html/GUID-BAA2DC07-D7DC-49BD-80A5-B4998B56B9BF.html)：RESTful 规范示例

__Go__

- [Go 语言设计与实现 | Go 语言设计与实现](https://draveness.me/golang/)
- [template - GoDoc](https://pkg.go.dev/text/template)：Go template 模版使用

__Python__

- [Python学习路线（学+测） - 阿里云大学](https://edu.aliyun.com/roadmap/python)
- [Python Cookbook 3rd Edition Documentation — python3-cookbook 3.0.0 文档](https://python3-cookbook.readthedocs.io/zh_CN/latest/index.html)

__Javascript__

- [认识 V8 引擎 - 知乎](https://zhuanlan.zhihu.com/p/27628685)

__算法__

- [十大经典排序算法（动图演示） - 一像素 - 博客园](https://www.cnblogs.com/onepixel/p/7674659.html)
- [30张图带你彻底理解红黑树 - 简书](https://www.jianshu.com/p/e136ec79235c)
- [动态规划 - 力扣（LeetCode）](https://leetcode-cn.com/tag/dynamic-programming/)
- [一个方法团灭 6 道股票问题 - 最佳买卖股票时机含冷冻期 - 力扣（LeetCode）](https://leetcode-cn.com/problems/best-time-to-buy-and-sell-stock-with-cooldown/solution/yi-ge-fang-fa-tuan-mie-6-dao-gu-piao-wen-ti-by-lab/)
- [labuladong的算法小抄 - labuladong的算法小抄](https://labuladong.gitbook.io/algo/)

### ** 面试必备 **

- [interview](https://interview.huihut.com)
- [CS-Notes](https://cyc2018.github.io/CS-Notes)

<!-- tabs:end -->
