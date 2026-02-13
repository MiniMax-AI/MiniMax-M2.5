# MiniMax M2.5 Model Transformers Deployment Guide

[English Version](./transformers_deploy_guide.md) | [Chinese Version](./transformers_deploy_guide_cn.md)

## Applicable Models

This document applies to the following models. You only need to change the model name during deployment.

- [MiniMaxAI/MiniMax-M2.5](https://huggingface.co/MiniMaxAI/MiniMax-M2.5)
- [MiniMaxAI/MiniMax-M2.1](https://huggingface.co/MiniMaxAI/MiniMax-M2.1)
- [MiniMaxAI/MiniMax-M2](https://huggingface.co/MiniMaxAI/MiniMax-M2)

The deployment process is illustrated below using MiniMax-M2.5 as an example.

## System Requirements

- OS: Linux

- Python: 3.9 - 3.12

- Transformers: 4.57.1

- GPU:

  - compute capability 7.0 or higher

  - Memory requirements: 220 GB for weights.

## Deployment with Python

It is recommended to use a virtual environment (such as **venv**, **conda**, or **uv**) to avoid dependency conflicts. 

We recommend installing Transformers in a fresh Python environment:

```bash
uv pip install transformers==4.57.1 torch accelerate --torch-backend=auto
```

Run the following Python script to run the model. Transformers will automatically download and cache the MiniMax-M2.5 model from Hugging Face.

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

## Common Issues

### Hugging Face Network Issues

If you encounter network issues, you can set up a proxy before pulling the model.

```bash
export HF_ENDPOINT=https://hf-mirror.com
```

### MiniMax-M2 model is not currently supported

Please check that trust_remote_code=True.

## Getting Support

If you encounter any issues while deploying the MiniMax model:

- Contact our technical support team through official channels such as email at [model@minimax.io](mailto:model@minimax.io)

- Submit an issue on our [GitHub](https://github.com/MiniMax-AI) repository

We continuously optimize the deployment experience for our models. Feedback is welcome!

