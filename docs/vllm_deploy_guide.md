# MiniMax M2.5 Model vLLM Deployment Guide

[English Version](./vllm_deploy_guide.md) | [Chinese Version](./vllm_deploy_guide_cn.md)

We recommend using [vLLM](https://docs.vllm.ai/en/stable/) to deploy the [MiniMax-M2.5](https://huggingface.co/MiniMaxAI/MiniMax-M2.5) model. vLLM is a high-performance inference engine with excellent serving throughput, efficient and intelligent memory management, powerful batch request processing capabilities, and deeply optimized underlying performance. We recommend reviewing vLLM's official documentation to check hardware compatibility before deployment.

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

We recommend installing vLLM in a fresh Python environment:

```bash
uv venv
source .venv/bin/activate
uv pip install vllm --torch-backend=auto
```

Run the following command to start the vLLM server. vLLM will automatically download and cache the MiniMax-M2.5 model from Hugging Face.

4-GPU deployment command:

```bash
SAFETENSORS_FAST_GPU=1 vllm serve \
    MiniMaxAI/MiniMax-M2.5 --trust-remote-code \
    --tensor-parallel-size 4 \
    --enable-auto-tool-choice --tool-call-parser minimax_m2 \
    --reasoning-parser minimax_m2_append_think
```

8-GPU deployment command:

```bash
SAFETENSORS_FAST_GPU=1 vllm serve \
    MiniMaxAI/MiniMax-M2.5 --trust-remote-code \
    --enable_expert_parallel --tensor-parallel-size 8 \
    --enable-auto-tool-choice --tool-call-parser minimax_m2 \
    --reasoning-parser minimax_m2_append_think 
```

## Testing Deployment

After startup, you can test the vLLM OpenAI-compatible API with the following command:

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

This vLLM version is outdated. Please upgrade to the latest version.

### torch.AcceleratorError: CUDA error: an illegal memory access was encountered
Add `--compilation-config "{\"cudagraph_mode\": \"PIECEWISE\"}"` to the startup parameters to resolve this issue. For example:

```bash
SAFETENSORS_FAST_GPU=1 vllm serve \
    MiniMaxAI/MiniMax-M2.5 --trust-remote-code \
    --enable_expert_parallel --tensor-parallel-size 8 \
    --enable-auto-tool-choice --tool-call-parser minimax_m2 \
    --reasoning-parser minimax_m2_append_think \
    --compilation-config "{\"cudagraph_mode\": \"PIECEWISE\"}"
```

### Output is garbled

If you encounter corrupted output when using vLLM to serve these models, you can upgrade to the nightly version (ensure it is a version after commit [cf3eacfe58fa9e745c2854782ada884a9f992cf7](https://github.com/vllm-project/vllm/commit/cf3eacfe58fa9e745c2854782ada884a9f992cf7))

## Getting Support

If you encounter any issues while deploying the MiniMax model:

- Contact our technical support team through official channels such as email at [model@minimax.io](mailto:model@minimax.io)

- Submit an issue on our [GitHub](https://github.com/MiniMax-AI) repository

We continuously optimize the deployment experience for our models. Feedback is welcome!
