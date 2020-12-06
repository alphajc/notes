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

