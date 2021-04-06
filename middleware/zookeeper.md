# Zookeeper

## 生成ACL

```bash
echo -n <user>:<password> | openssl dgst -binary -sha1 | openssl base64
```
