# MinIO

## 使用docker启动Minio

```shell
docker run --name myMinio \
-p 9000:9000 \
-p 9001:9001 \
-e "MINIO_ROOT_USER=chuxin" \
-e "MINIO_ROOT_PASSWORD=12345678" \
-v /Users/chuxin/minio/data:/data \
bitnami/minio \
/opt/bitnami/minio/bin/minio server /data --console-address ":9001" --address ":9000"
```

## 视频+封面URL

