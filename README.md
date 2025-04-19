# [vLLM](https://docs.vllm.ai) in [Docker](https://www.docker.com)

## Prepare

```shell
cp tmp.env .env
```

Fill in the required fields like `HUGGING_FACE_HUB_TOKEN`.

## Run

```shell
docker compose up -d
```

## Check

```shell
curl -X POST "http://localhost:8080/vllms/qwen2.5-0.5b/v1/chat/completions" \
	-H "Content-Type: application/json" \
	--data '{
		"model": "Qwen/Qwen2.5-0.5B-Instruct-GPTQ-Int4",
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
