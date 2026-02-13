# MiniMax M2.5 模型 Transformers 部署指南

[英文版](./transformers_deploy_guide.md) | [中文版](./transformers_deploy_guide_cn.md)

## 本文档适用模型

本文档适用以下模型，只需在部署时修改模型名称即可。

- [MiniMaxAI/MiniMax-M2.5](https://huggingface.co/MiniMaxAI/MiniMax-M2.5)
- [MiniMaxAI/MiniMax-M2.1](https://huggingface.co/MiniMaxAI/MiniMax-M2.1)
- [MiniMaxAI/MiniMax-M2](https://huggingface.co/MiniMaxAI/MiniMax-M2)

以下以 MiniMax-M2.5 为例说明部署流程。

## 环境要求

- OS：Linux

- Python：3.9 - 3.12

- Transformers: 4.57.1

- GPU：

  - compute capability 7.0 or higher

  - 显存需求：权重需要 220 GB

## 使用 Python 部署

建议使用虚拟环境（如 **venv**、**conda**、**uv**）以避免依赖冲突。

建议在全新的 Python 环境中安装 Transformers:

```bash
uv pip install transformers==4.57.1 torch accelerate --torch-backend=auto
```

运行如下 Python 命令运行模型，Transformers 会自动从 Huggingface 下载并缓存 MiniMax-M2.5 模型。

```python
from transformers import AutoModelForCausalLM, AutoTokenizer, GenerationConfig
import torch

MODEL_PATH = "MiniMaxAI/MiniMax-M2.5"

model = AutoModelForCausalLM.from_pretrained(
    MODEL_PATH,
    device_map="auto",
    trust_remote_code=True,
)
tokenizer = AutoTokenizer.from_pretrained(MODEL_PATH)

messages = [
    {"role": "user", "content": [{"type": "text", "text": "What is your favourite condiment?"}]},
    {"role": "assistant", "content": [{"type": "text", "text": "Well, I'm quite partial to a good squeeze of fresh lemon juice. It adds just the right amount of zesty flavour to whatever I'm cooking up in the kitchen!"}]},
    {"role": "user", "content": [{"type": "text", "text": "Do you have mayonnaise recipes?"}]}
]

model_inputs = tokenizer.apply_chat_template(messages, return_tensors="pt", add_generation_prompt=True).to("cuda")

generated_ids = model.generate(model_inputs, max_new_tokens=100, generation_config=model.generation_config)

response = tokenizer.batch_decode(generated_ids)[0]

print(response)
```

## 常见问题

### Huggingface 网络问题

如果遇到网络问题，可以设置代理后再进行拉取。

```bash
export HF_ENDPOINT=https://hf-mirror.com
```

### MiniMax-M2 model is not currently supported

请确认开启 trust_remote_code=True。

## 获取支持

如果在部署 MiniMax 模型过程中遇到任何问题：

- 通过邮箱 [model@minimax.io](mailto:model@minimax.io) 等官方渠道联系我们的技术支持团队

- 在我们的 [GitHub](https://github.com/MiniMax-AI) 仓库提交 Issue

- 通过我们的 [官方企业微信交流群](https://github.com/MiniMax-AI/MiniMax-AI.github.io/blob/main/images/wechat-qrcode.jpeg) 反馈

我们会持续优化模型的部署体验，欢迎反馈！
