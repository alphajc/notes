# 系统安全

## 证书

### Openssl 操作证书

#### 签发证书

```bash
openssl genrsa -out private/demo.key 2048 # 生成私钥
openssl req -new -key private/demo.key -subj "/CN=demo" -out private/demo.csr # 生成证书请求
openssl x509 -req -in private/demo.csr -CA ca.crt -CAkey private/ca.key -CAcreateserial -out demo.crt -days 1000 -extensions v3_req # 使用根证书颁发证书
```

#### 签发带扩展的证书

* 扩展文本内容(`demo.ext`)

    ```ini
    subjectAltName = DNS:k8s1.demo.com,DNS:k8s.demo.com,DNS:kubernetes,DNS:kubernetes.default,DNS:kubernetes.default.svc, DNS:kubernetes.default.svc.cluster.local, IP:10.96.0.1, IP:172.21.0.8, IP:123.206.95.30
    ```

* 签发：

    ```bash
    openssl x509 -req -in private/demo.csr -CA ca.crt -CAkey private/ca.key -CAcreateserial -out demo.crt -days 1000 -extfile demo.ext
    ```

#### 查看证书内容

```bash
openssl x509 -in demo.crt -noout -text
```

#### 自签证书根证书

```bash
openssl genrsa -aes256 -out private/ca.key 1024
openssl req -new -key private/ca.key -out private/ca.csr -subj "/C=CN/ST=myprovince/L=mycity/O=myorganization/OU=mygroup/CN=myname"
```
