# Istio

## 常见问题

从[官方问题库](https://istio.io/latest/zh/docs/ops/common-problems/)着手解决所遇到的问题

### 503 问题

在 sidecar 的访问日志中，出现 response-flag 为 `UC` 的日志，伴随着连接被上游断开报错信息的出现

#### 参考链接

* Istio：503、UC 和 TCP：https://cloud.tencent.com/developer/article/1471457
* Istio 下的优雅启动和优雅关闭：https://www.dozer.cc/2020/02/graceful-start-and-shutdown.html
* Envoy 管理接口：https://www.envoyproxy.io/docs/envoy/latest/operations/admin