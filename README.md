# FastChat on Mac (Apple Silicon)

https://github.com/lm-sys/FastChat on Mac

## How to run OpenAI compatible FastChat server on Mac (Apple Silicon) with its Metal GPU

Download the `docker-compose.yaml` and run

```
wget https://github.com/making/fastchat-on-mac/raw/main/docker-compose.yaml
docker-compose up
```

It launches controller, gradio (Web UI, optional), and openai api server. These do not use GPU, so it is easy to start them with docker.

Then

```
pip3 install "fschat[model_worker,webui]"
```

Start the model worker outside of docker and specify the `--device mps` option to take advantage of Metal GPU.

```
python3 -m fastchat.serve.model_worker --port 21002 --model-path lmsys/vicuna-7b-v1.3 --device mps --load-8bit --controller-address http://localhost:21001 --worker-address http://host.docker.internal:21002
```

* http://localhost:7860 WebUI
* http://localhost:8000 OpenAI API


## Access the OpenAI API

https://github.com/lm-sys/FastChat/blob/main/docs/openai_api.md

```
$ curl -s http://localhost:8000/v1/chat/completions \
  -H "Content-Type: application/json" \
  -d '{
    "model": "vicuna-7b-v1.3",
    "messages": [{"role": "user", "content": "Tell me a joke"}]
  }'

{
  "id": "chatcmpl-fiy5p8FeTLjUNdfL2yS28q",
  "object": "chat.completion",
  "created": 1695969537,
  "model": "vicuna-7b-v1.3",
  "choices": [
    {
      "index": 0,
      "message": {
        "role": "assistant",
        "content": "Sure, here's a joke for you:\n\nWhy don't scientists trust atoms?\n\nBecause they make up everything!"
      },
      "finish_reason": "stop"
    }
  ],
  "usage": {
    "prompt_tokens": 44,
    "total_tokens": 75,
    "completion_tokens": 31
  }
}
```


## Run the Spring AI OpenAI demo

```
git clone https://github.com/rd-1-2022/ai-openai-helloworld.git
cd ai-openai-helloworld
./mvnw clean package -DskipTests

java -jar target/ai-openai-helloworld-0.0.1-SNAPSHOT.jar --spring.ai.openai.base-url=http://localhost:8000 --spring.ai.openai.api-key=dummy --spring.ai.openai.model=vicuna-7b-v1.3
```


```
$ curl -s --get --data-urlencode 'message=Tell me a joke about a cow.' http://localhost:8080/ai/simple | jq .
{
  "completion": "Why couldn't the cow play in the band?\n\nBecause it was too moo-sical!"
}
```

## How to build the docker image

```
pack build ghcr.io/making/fastchat --builder paketobuildpacks/builder:base --path image --publish
```