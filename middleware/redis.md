# Redis

## 常用命令

### 查看每客户端连接数 <!-- {docsify-ignore} -->

```bash
redis-cli client list | awk -F '[=:]' '{sum[$2] += 1} END {for(i in sum) print i, sum[i];}' | sort -n -r -k 2
```
