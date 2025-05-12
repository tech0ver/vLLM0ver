# vLLM in Docker

Run [vLLM](https://docs.vllm.ai) with [Docker Compose](https://docs.docker.com/compose/).

> [!NOTE]
> For Kubernetes, see the [vLLM production stack](https://github.com/vllm-project/production-stack).

## Usage

### Prepare

```shell
cp tmp.env .env
```

Set `HUGGING_FACE_HUB_TOKEN` and other required fields.

### Run

```shell
docker compose up -d
```

### Test

```shell
curl -X POST "http://localhost:8080/vllms/qwen2.5-0.5b/v1/chat/completions" \
  -H "Content-Type: application/json" \
  --data '{
    "messages": [
      {
        "role": "user",
        "content": "What is the capital of France?"
      }
    ]
  }'
```

## Add Model

> [!IMPORTANT]
> Refer to [supported models](https://docs.vllm.ai/en/latest/models/supported_models.html)
> and [engine arguments](https://docs.vllm.ai/en/latest/serving/engine_args.html).

### Edit [docker-compose.yml](docker-compose.yml)

```diff
...
services:
  proxy: <...>

+ deepseek-r1:
+   <<: *base-vllm
+   command:
+     - --task=generate
+     - --model=jakiAJK/DeepSeek-R1-Distill-Qwen-1.5B_GPTQ-int4
+     - --quantization=gptq
+     - --dtype=float16
+     - --max-model-len=65536
+     - --gpu-memory-utilization=0.8
...
```

### Test Model

```shell
curl -X POST "http://localhost:8080/vllms/deepseek-r1/v1/chat/completions" \
  -H "Content-Type: application/json" \
  --data '{
    "messages": [
      {
        "role": "user",
        "content": "What is the capital of France?"
      }
    ]
  }'
```

### Update [prometheus.yml](docker/prometheus.yml)

```diff
...
scrape_configs:
  - job_name: vllm-job
    static_configs:
      - targets:
          - qwen2.5-0.5b:8000
          - user-bge-m3:8000
+         - deepseek-r1:8000
...
```

## Basic Auth

### Create Credentials

```shell
htpasswd -cb docker/nginx.htpasswd YOUR_USER YOUR_PASSWORD
```

### Update [docker-compose.yml](docker-compose.yml)

```diff
...
services:
  proxy:
    image: nginx:${NGINX_TAG:-1.27-alpine}
    volumes:
      - ./docker/nginx.conf:/etc/nginx/nginx.conf:ro
+     - ./docker/nginx.htpasswd:/etc/nginx/.htpasswd:ro
    ports:
      - 8080:8080
...
```

### Update [nginx.conf](docker/nginx.conf)

```diff
...
http {
    resolver 127.0.0.11 valid=10s;

    server {
        listen 8080;

+       auth_basic "Restricted Area";
+       auth_basic_user_file /etc/nginx/.htpasswd;

        location ~ ^/vllms/(?<model>[^/]+?)/(?<path>.*)$ {
...
```

## Scaling

### Update [docker-compose.yml](docker-compose.yml)

```diff
...
services:
  proxy: <...>

  qwen2.5-0.5b:
    <<: *base-vllm
+   scale: 2
    command:
      - --task=generate
      - --model=Qwen/Qwen2.5-0.5B-Instruct-GPTQ-Int4
...
```

## Observability

### Launch

```shell
docker compose -f docker-compose.obs.yml up -d
```

### Access

Go to [http://localhost:3000](http://localhost:3000) (login: `admin`/`admin`).
