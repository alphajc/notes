# 运维

## Linux 性能调优

### 高负载排查

* `load average`：测量正在运行和等待运行的线程数（CPU，磁盘，不间断锁）。换句话说，它测量不完全空闲的线程数。[_引自_](https://zhuanlan.zhihu.com/p/75975041)

* 诱因：__CPU__ 、__磁盘__ 、__某些锁__

* 查看负载情况

    * uptime/w命令：

        ```
        $ uptime
         07:04:25 up 50 days,  5:54,  1 user,  load average: 0.01, 0.02, 0.00
        $ w
         07:04:26 up 50 days,  5:55,  1 user,  load average: 0.00, 0.00, 0.00
        USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
        ubuntu   pts/0    11.99.99.234     07:04    2.00s  0.03s  0.00s w
        ```

    * top命令：

        ```
        top - 07:06:21 up 50 days,  5:57,  1 user,  load average: 0.00, 0.00, 0.00
        Tasks: 102 total,   1 running, 101 sleeping,   0 stopped,   0 zombie
        %Cpu(s):  0.3 us,  0.0 sy,  0.0 ni, 99.7 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
        MiB Mem :    978.6 total,    150.4 free,    204.9 used,    623.3 buff/cache
        MiB Swap:      0.0 total,      0.0 free,      0.0 used.    620.6 avail Mem

        ...
        ```

    > 说明：  
    > `load average`的三列分别表示1分钟、5分钟、15分钟的负载值。  
    > 在1核的情况下，`load average` 理想值为1，说明CPU工作量较饱和； 超过1，说明CPU过载，有等待情况；小于1说明CPU较闲，浪费资源。  
    > `load average`理想值为CPU核心数

* 使用工具

    * cpu工具：top、mpstat（查看处理器cpu利用率）、pidstat（查看进程cpu利用率）
    * io工具：iotop


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

