# Argo

## Argocd

### 重置 argocd 密码

```bash
kubectl -n argocd patch secret argocd-secret  -p '{"data": {"admin.password": null, "admin.passwordMtime": null}}' # 清除密码
kubectl -n argocd delete pod -l app.kubernetes.io/name=argocd-server # 重建 pod
```

重置后的密码为新的 pod 的名字

```bash
kubectl get pods -n argocd -l app.kubernetes.io/name=argocd-server -o name | cut -d'/' -f 2
```
