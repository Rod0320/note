# 拉取镜像

```shell
docker pull luode0320/chatgpt:1.0
```

# 启动

```shell
docker run -d -p 3000:3000 \
    --privileged=true \
    --restart=always  \
   -e OPENAI_API_KEY="sk-xxx" \
   -e CODE="luode" \
   luode0320/chatgpt:1.0
```

