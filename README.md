# LFX-Mentorship-WasmEdge-2024-01-Pre-test

## Environment

```bash
uname -a
Darwin WXQ6ML73V1 23.2.0 Darwin Kernel Version 23.2.0: Wed Nov 15 21:53:18 PST 2023; root:xnu-10002.61.3~2/RELEASE_ARM64_T6000 arm64
```

## Build WasmEdge with WASI-NN llama.cpp Backend

Install dependencies.

```bash
brew install cmake
brew install grpc
brew install llvm
```

build and install WasmEdge from source directly on the macOS arm64 platform.

```bash
git clone https://github.com/WasmEdge/WasmEdge.git -b hydai/0.13.5_ggml_lts 
cd WasmEdge

# Apple Silicon Model
export LLVM_DIR=/opt/homebrew/opt/llvm/lib/cmake
cmake -GNinja -Bbuild -DCMAKE_BUILD_TYPE=Release \
  -DWASMEDGE_PLUGIN_WASI_NN_BACKEND="GGML" \
  -DWASMEDGE_PLUGIN_WASI_NN_GGML_LLAMA_METAL=ON \
  -DWASMEDGE_PLUGIN_WASI_NN_GGML_LLAMA_BLAS=OFF \
  .
```

![build-wasmedge-with-wasi-nn-llama.cpp-backend.jpg](./image/build-wasmedge-with-wasi-nn-llama.cpp-backend.jpg)

```bash
cmake --build build
```

![build-wasmedge](./image/build-wasmedge.jpg)


```bash
cmake --install build
```

![install-wasmedge](./image/install-wasmedge.jpg)

## Create an OpenAI compatible API server for your LLM

### Get the llama-api-server.wasm app

```bash
curl -LO https://github.com/second-state/LlamaEdge/releases/latest/download/llama-api-server.wasm
```

### Download the model

```bash
curl -LO https://huggingface.co/second-state/Llama-2-7B-Chat-GGUF/resolve/main/Llama-2-7b-chat-hf-Q5_K_M.gguf
```

### Run the API server via curl

```bash
wasmedge --dir .:. --nn-preload default:GGML:AUTO:llama-2-7b-chat.Q5_K_M.gguf llama-api-server.wasm -p llama-2-chat
```

![run-api-server-without-ui](./image/run-api-server-without-ui.jpg)


### Test the API server via terminal

List models.

```bash
curl -X POST http://localhost:8080/v1/models -H 'accept:application/json' -s | jq

# Response
{
  "object": "list",
  "data": [
    {
      "id": "default:llama-2-chat",
      "created": 1706836242,
      "object": "model",
      "owned_by": "Not specified"
    }
  ]
}
```

Ask a question using OpenAI's JSON message format.

```bash
curl -X POST http://localhost:8080/v1/chat/completions \
-H 'accept:application/json' -H 'Content-Type: application/json' \
-d '{
  "messages": [
    {
      "role": "system",
      "content": "You are a helpful assistant."
    },
    {
      "role": "user",
      "content": "Who is Robert Oppenheimer?"
    }
  ],
  "model": "llama-2-chat"
}' -s | jq

# Response
{
  "id": "d74ff381-6830-43dd-8f46-9f9815b3f62d",
  "object": "chat.completion",
  "created": 1706837616,
  "model": "llama-2-chat",
  "choices": [
    {
      "index": 0,
      "message": {
        "role": "assistant",
        "content": "Ah, an excellent question! Robert Oppenheimer (1904-1967) was a renowned American physicist and science administrator who played a crucial role in the development of the atomic bomb during World War II. He is often referred to as the \"Father of the Atomic Bomb.\"\nOppenheimer was born in New York City and grew up in a family that valued education and intellectual pursuits. He showed an early interest in science and mathematics, and"
      },
      "finish_reason": "stop"
    }
  ],
  "usage": {
    "prompt_tokens": 34,
    "completion_tokens": 103,
    "total_tokens": 137
  }
}
```

The following command sends a prompt to the API server and gets the completion:

```bash
 curl -X POST http://localhost:8080/v1/completions \
    -H 'accept:application/json' \
    -H 'Content-Type: application/json' \
    -d '{"prompt":["Long long ago, "], "model":"tinyllama"}' -s | jq
    
# Response
{
  "id": "e705172e-cf83-4c87-b331-074fd7136ace",
  "choices": [
    {
      "finish_reason": "stop",
      "index": 0,
      "logprobs": null,
      "text": "in a far-off land, there was an emperor who ruled over his people with great wisdom and kindness. He was loved by all, and he knew that his people were the key to the prosperity of his kingdom. So he decided to hold a great contest, to find the most talented and capable person in his empire.\nThe contest was announced throughout the land, and people from far and wide came to the capital city to compete. The emperor set up a special arena"
    }
  ],
  "created": 1706837693,
  "model": "tinyllama",
  "object": "text_completion",
  "usage": {
    "prompt_tokens": 3,
    "completion_tokens": 84,
    "total_tokens": 87
  }
}

```

### Add a web UI

Download front-end Web UI and extract it.

```bash
curl -LO https://github.com/second-state/chatbot-ui/releases/latest/download/chatbot-ui.tar.gz
tar xzf chatbot-ui.tar.gz
```

Use the same command line to create the API server.

```bash
wasmedge --dir .:. --nn-preload default:GGML:AUTO:llama-2-7b-chat.Q5_K_M.gguf llama-api-server.wasm -p llama-2-chat
```

![run-api-server-with-ui](./image/run-api-server-with-ui.jpg)

Open http://127.0.0.1:8080 from the browser.

![access-web-ui](./image/access-web-ui.jpg)
