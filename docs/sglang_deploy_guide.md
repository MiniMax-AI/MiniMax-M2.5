# MiniMax M2.5 Model SGLang Deployment Guide

[English Version](./sglang_deploy_guide.md) | [Chinese Version](./sglang_deploy_guide_cn.md)

We recommend using [SGLang](https://github.com/sgl-project/sglang) to deploy the [MiniMax-M2.5](https://huggingface.co/MiniMaxAI/MiniMax-M2.5) model. SGLang is a high-performance inference engine with excellent serving throughput, efficient and intelligent memory management, powerful batch request processing capabilities, and deeply optimized underlying performance. We recommend reviewing SGLang's official documentation to check hardware compatibility before deployment.

## Applicable Models

This document applies to the following models. You only need to change the model name during deployment.

- [MiniMaxAI/MiniMax-M2.5](https://huggingface.co/MiniMaxAI/MiniMax-M2.5)
- [MiniMaxAI/MiniMax-M2.1](https://huggingface.co/MiniMaxAI/MiniMax-M2.1)
- [MiniMaxAI/MiniMax-M2](https://huggingface.co/MiniMaxAI/MiniMax-M2)

The deployment process is illustrated below using MiniMax-M2.5 as an example.

## System Requirements

- OS: Linux

- Python: 3.9 - 3.12

- GPU:

  - compute capability 7.0 or higher

  - Memory requirements: 220 GB for weights, 240 GB per 1M context tokens

The following are recommended configurations; actual requirements should be adjusted based on your use case:

- **96G x4** GPU: Supports a total KV Cache capacity of 400K tokens.

- **144G x8** GPU: Supports a total KV Cache capacity of up to 3M tokens.

> **Note**: The values above represent the total aggregate hardware KV Cache capacity. The maximum context length per individual sequence remains **196K** tokens.

## Deployment with Python

It is recommended to use a virtual environment (such as **venv**, **conda**, or **uv**) to avoid dependency conflicts. 

We recommend installing SGLang in a fresh Python environment:

```bash
uv venv
source .venv/bin/activate
uv pip install sglang
```

Run the following command to start the SGLang server. SGLang will automatically download and cache the MiniMax-M2.5 model from Hugging Face.

4-GPU deployment command:

```bash
python -m sglang.launch_server \
    --model-path MiniMaxAI/MiniMax-M2.5 \
    --tp-size 4 \
    --tool-call-parser minimax-m2 \
    --reasoning-parser minimax-append-think \
    --host 0.0.0.0 \
    --trust-remote-code \
    --port 8000 \
    --mem-fraction-static 0.85
```

8-GPU deployment command:

```bash
python -m sglang.launch_server \
    --model-path MiniMaxAI/MiniMax-M2.5 \
    --tp-size 8 \
    --ep-size 8 \
    --tool-call-parser minimax-m2 \
    --trust-remote-code \
    --host 0.0.0.0 \
    --reasoning-parser minimax-append-think \
    --port 8000 \
    --mem-fraction-static 0.85
```

## Testing Deployment

After startup, you can test the SGLang OpenAI-compatible API with the following command:

```bash
curl http://localhost:8000/v1/chat/completions \
    -H "Content-Type: application/json" \
    -d '{
        "model": "MiniMaxAI/MiniMax-M2.5",
        "messages": [
            {"role": "system", "content": [{"type": "text", "text": "You are a helpful assistant."}]},
            {"role": "user", "content": [{"type": "text", "text": "Who won the world series in 2020?"}]}
        ]
    }'
```

## Common Issues

### MiniMax-M2 model is not currently supported

Please upgrade to the latest stable version, >= v0.5.4.post1.

## Getting Support

If you encounter any issues while deploying the MiniMax model:

- Contact our technical support team through official channels such as email at [model@minimax.io](mailto:model@minimax.io)

- Submit an issue on our [GitHub](https://github.com/MiniMax-AI) repository

We continuously optimize the deployment experience for our models. Feedback is welcome!

