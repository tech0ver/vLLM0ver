# [vLLM](https://docs.vllm.ai) in [Docker](https://www.docker.com)

## Using

### Prepare

```shell
cp tmp.env .env
```

Fill in the required fields like `HUGGING_FACE_HUB_TOKEN`.

### Run

```shell
docker compose up -d
```

### Check

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

## Adding new model

> [!IMPORTANT]
> See [supported models](https://docs.vllm.ai/en/latest/models/supported_models.html)
> and [arguments](https://docs.vllm.ai/en/latest/serving/engine_args.html)
> by vLLM.

### Update [docker-compose.yml](docker-compose.yml)

```diff
...
services:
  proxy:
    image: nginx:${NGINX_TAG:-1.27-alpine}
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf:ro
    ports:
      - 8080:8080

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

### Check new model

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

## Basic Auth

### Create `nginx.htpasswd`

```shell
htpasswd -cb nginx.htpasswd YOUR_USER YOUR_PASSWORD
```

### Update [docker-compose.yml](docker-compose.yml)

```diff
...
services:
  proxy:
    image: nginx:${NGINX_TAG:-1.27-alpine}
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf:ro
+     - ./nginx.htpasswd:/etc/nginx/.htpasswd:ro
    ports:
      - 8080:8080
...
```

### Update [nginx.conf](nginx.conf)

```diff
events {}

http {
    resolver 127.0.0.11 valid=10s;

    server {
        listen 8080;

+       auth_basic "Restricted Area";
+       auth_basic_user_file /etc/nginx/.htpasswd;

        location ~ ^/vllms/(?<model>[^/]+?)/(?<path>.*)$ {
...
```

## Observability

### Run

```shell
docker compose -f docker-compose.obs.yml up -d
```

### Open

Go to http://localhost:3000 as `admin`/`admin`.
