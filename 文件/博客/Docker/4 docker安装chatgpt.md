# 拉取镜像

```shell
docker pull luode0320/chatgpt:1.0
```

# 启动

```shell
docker run -d -p 3000:3000 \
    --privileged=true \
    --restart=always  \
   -e OPENAI_API_KEY="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" \
   -e CODE="luode" \
   luode0320/chatgpt:1.0
```

