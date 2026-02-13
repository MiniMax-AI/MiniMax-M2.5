# MiniMax M2.5 模型 vLLM 部署指南

[英文版](./vllm_deploy_guide.md) | [中文版](./vllm_deploy_guide_cn.md)

我们推荐使用 [vLLM](https://docs.vllm.ai/en/stable/) 来部署 [MiniMax-M2.5](https://huggingface.co/MiniMaxAI/MiniMax-M2.5) 模型。vLLM 是一个高性能的推理引擎，其具有卓越的服务吞吐、高效智能的内存管理机制、强大的批量请求处理能力、深度优化的底层性能等特性。我们建议在部署之前查看 vLLM 的官方文档以检查硬件兼容性。

## 本文档适用模型

本文档适用以下模型，只需在部署时修改模型名称即可。

- [MiniMaxAI/MiniMax-M2.5](https://huggingface.co/MiniMaxAI/MiniMax-M2.5)
- [MiniMaxAI/MiniMax-M2.1](https://huggingface.co/MiniMaxAI/MiniMax-M2.1)
- [MiniMaxAI/MiniMax-M2](https://huggingface.co/MiniMaxAI/MiniMax-M2)

以下以 MiniMax-M2.5 为例说明部署流程。

## 环境要求

- OS：Linux

- Python：3.9 - 3.12

- GPU：

  - compute capability 7.0 or higher

  - 显存需求：权重需要 220 GB，每 1M 上下文 token 需要 240 GB

以下为推荐配置，实际需求请根据业务场景调整：

- **96G x4 GPU**：总 KV Cache 容量支持 40 万 token。

- **144G x8 GPU**：总 KV Cache 容量支持高达 300 万 token。

> **注**：以上数值为硬件支持的最大并发缓存总量，模型单序列（Single Sequence）长度上限仍为 196k。

## 使用 Python 部署

建议使用虚拟环境（如 **venv**、**conda**、**uv**）以避免依赖冲突。

建议在全新的 Python 环境中安装 vLLM：

```bash
uv venv
source .venv/bin/activate
uv pip install vllm --torch-backend=auto
```

运行如下命令启动 vLLM 服务器，vLLM 会自动从 Huggingface 下载并缓存 MiniMax-M2.5 模型。

4 卡部署命令：

```bash
SAFETENSORS_FAST_GPU=1 vllm serve \
    MiniMaxAI/MiniMax-M2.5 --trust-remote-code \
    --tensor-parallel-size 4 \
    --enable-auto-tool-choice --tool-call-parser minimax_m2 \
    --reasoning-parser minimax_m2_append_think
```

8 卡部署命令：

```bash
SAFETENSORS_FAST_GPU=1 vllm serve \
    MiniMaxAI/MiniMax-M2.5 --trust-remote-code \
    --enable_expert_parallel --tensor-parallel-size 8 \
    --enable-auto-tool-choice --tool-call-parser minimax_m2 \
    --reasoning-parser minimax_m2_append_think 
```

## 测试部署

启动后，可以通过如下命令测试 vLLM OpenAI 兼容接口：

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

## 常见问题

### Huggingface 网络问题

如果遇到网络问题，可以设置代理后再进行拉取。

```bash
export HF_ENDPOINT=https://hf-mirror.com
```

### MiniMax-M2 model is not currently supported

该 vLLM 版本过旧，请升级到最新版本。

### torch.AcceleratorError: CUDA error: an illegal memory access was encountered
在启动参数添加 `--compilation-config "{\"cudagraph_mode\": \"PIECEWISE\"}"` 可以解决。例如：

```bash
SAFETENSORS_FAST_GPU=1 vllm serve \
    MiniMaxAI/MiniMax-M2.5 --trust-remote-code \
    --enable_expert_parallel --tensor-parallel-size 8 \
    --enable-auto-tool-choice --tool-call-parser minimax_m2 \
    --reasoning-parser minimax_m2_append_think \
    --compilation-config "{\"cudagraph_mode\": \"PIECEWISE\"}"
```

### 模型输出乱码

如果您在使用 vLLM 运行这些模型时遇到输出乱码，可以升级到最新版本（请至少确保版本在提交 [cf3eacfe58fa9e745c2854782ada884a9f992cf7](https://github.com/vllm-project/vllm/commit/cf3eacfe58fa9e745c2854782ada884a9f992cf7) 之后）。

## 获取支持

如果在部署 MiniMax 模型过程中遇到任何问题：

- 通过邮箱 [model@minimax.io](mailto:model@minimax.io) 等官方渠道联系我们的技术支持团队

- 在我们的 [GitHub](https://github.com/MiniMax-AI) 仓库提交 Issue

- 通过我们的 [官方企业微信交流群](https://github.com/MiniMax-AI/MiniMax-AI.github.io/blob/main/images/wechat-qrcode.jpeg) 反馈

我们会持续优化模型的部署体验，欢迎反馈！
