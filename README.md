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
curl -X POST "http://localhost:8080/models/qwen2.5-0.5b/v1/chat/completions" \
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
