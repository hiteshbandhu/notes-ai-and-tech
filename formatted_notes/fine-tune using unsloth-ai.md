---
tags: [AI, LLM, Fine-tuning, Unsloth, TinyLlama, LoRA, QLoRA, WandB, MachineLearning, DeepLearning, Python]
---

**Context:** This note summarizes an article on fine-tuning a Tiny-Llama model using Unsloth, covering concepts like LoRA, QLoRA, and integrating with Weights & Biases (WandB) for logging. It provides a practical guide for efficient LLM fine-tuning on consumer hardware.


Title: Fine-tuning A Tiny-Llama Model with Unsloth

URL Source: https://www.analyticsvidhya.com/blog/2024/02/fine-tuning-a-tiny-llama-model-with-unsloth/

Published Time: 2024-02-02T11:45:04+00:00

Markdown Content:
Introduction
------------

After the Llama and Mistral models were released, the open-source LLMs took the limelight out of OpenAI. Since then, multiple models have been released based on Llama and Mistral architecture, performing on par with proprietary models like GPT-3.5 Turbo, Claude, Gemini, etc. However, these models are too large to be used in consumer hardware.

But lately, there has been an emergence of a new class of LLMs. These are the LLMs in the sub-7B parameter category. Fewer parameters make them compact enough to be run in consumer hardware while keeping efficiency comparable to the 7B models. Models like Tiny-Llama-1B, Microsoft’s Phi-2, and Alibaba’s Qwen-3b can be great substitutes for larger models to run locally or deploy on edge. At the same time, fine-tuning is crucial to bring the best out of any base model for any downstream tasks.  
Here, we will explore how to Fine-tune a base [Tiny-Llama model](https://www.analyticsvidhya.com/blog/2024/01/tinyllama-b-size-doesnt-matter/) on a cleaned Alpaca dataset.

![Image 1: Fine-tuning A Tiny-Llama](https://cdn.analyticsvidhya.com/wp-content/uploads/2024/02/image-23.png)

#### Learning Objectives

*   Understand fine-tuning and different methods of it.
*   Learn about tools and techniques for efficient fine-tuning.
*   Learn about WandB for logging training logs.
*   Fine-tune Tiny-Llama on the Alpaca dataset in Colab.

_**This article was published as a part of the**_ [_**Data Science Blogathon.**_](https://datahack.analyticsvidhya.com/blogathon/)

Table of contents
-----------------

*   [What is LLM Fine-Tuning?](https://www.analyticsvidhya.com/blog/2024/02/fine-tuning-a-tiny-llama-model-with-unsloth/#h-what-is-llm-fine-tuning)
*   [Fine-Tuning with Unsloth](https://www.analyticsvidhya.com/blog/2024/02/fine-tuning-a-tiny-llama-model-with-unsloth/#h-fine-tuning-with-unsloth)
*   [Logging with WandB](https://www.analyticsvidhya.com/blog/2024/02/fine-tuning-a-tiny-llama-model-with-unsloth/#h-logging-with-wandb)
*   [How to Fine-tune Tiny-Llama?](https://www.analyticsvidhya.com/blog/2024/02/fine-tuning-a-tiny-llama-model-with-unsloth/#h-how-to-fine-tune-tiny-llama)
    *   [Prepare Data](https://www.analyticsvidhya.com/blog/2024/02/fine-tuning-a-tiny-llama-model-with-unsloth/#h-prepare-data)
    *   [Configure WandB](https://www.analyticsvidhya.com/blog/2024/02/fine-tuning-a-tiny-llama-model-with-unsloth/#h-configure-wandb)
    *   [Train Model](https://www.analyticsvidhya.com/blog/2024/02/fine-tuning-a-tiny-llama-model-with-unsloth/#h-train-model)
    *   [Inferencing](https://www.analyticsvidhya.com/blog/2024/02/fine-tuning-a-tiny-llama-model-with-unsloth/#h-inferencing)
*   [Frequently Asked Questions](https://www.analyticsvidhya.com/blog/2024/02/fine-tuning-a-tiny-llama-model-with-unsloth/#h-frequently-asked-questions)

What is LLM Fine-Tuning?
------------------------

Fine-tuning is the process of making a pre-trained model learn new knowledge. The pre-trained model is a general-purpose model trained on a large amount of data. However, in most cases, they fail to perform as intended, and fine-tuning is the most effective way to make the model adapt to specific use cases. For example, base [LLMs](https://www.analyticsvidhya.com/blog/2023/03/an-introduction-to-large-language-models-llms/) do well at text generation on single-turn QA but struggle with multi-turn conversations like chat models.

The base models need to be trained on transcripts of dialogues to be able to hold multi-turn conversations. Fine-tuning is essential to mold pre-trained models into different avatars. The quality of Fine-tuned models depends on the quality of data and base model capabilities. There are multiple ways to model fine-tuning, like LoRA, QLoRA, etc.

Let’s briefly go through these concepts.

#### LoRA

LoRA stands for Low-rank Adaptation, a popular fine-tuning technique in which we select a few trainable parameters instead of updating all the parameters via a low-rank approximation of original weight matrices. The LoRA model can be Fine-tuned faster on less compute-intensive hardware.

#### QLoRA

QLoRA or Quantized LoRA is a step further than the LoRA. Instead of a full-precision model, it quantizes the model weights to lower floating point precision before applying LoRA. Quantization is the process of downcasting higher bit values to lower values. A 4-bit quantization process involves quantizing the 16-bit weights to 4-bit float values.

Quantizing the model leads to a substantial reduction in model size with comparable accuracy to the original model. In QLoRA, we take a quantized model and apply LoRA to it. The models can be quantized in multiple ways, such as through llama.cpp, AWQ, bitsandbytes, etc.

Fine-Tuning with Unsloth
------------------------

Unsloth is an open-source platform for fine-tuning popular Large Language Models faster. It supports popular LLMs, including Llama-2 and Mistral, and their derivatives like Yi, Open-hermes, etc. It implements custom triton kernels and a manual back-prop engine to improve the speed of the model training.

Here, we will use the Unsloth to Fine-tune a base 4-bit quantized Tiny-Llama model on the [Alpaca](https://huggingface.co/datasets/yahma/alpaca-cleaned) dataset. The model is quantized with bits and bytes, and kernels are optimized with OpenAI’s Triton.

![Image 2: Unsloth](https://cdn.analyticsvidhya.com/wp-content/uploads/2024/02/image-24.png)

Logging with WandB
------------------

In Machine learning, it is crucial to log training and evaluation metrics. This gives us a complete picture of the train run. [Weights and Biases](https://docs.wandb.ai/) (WandB) is an open-source library for visualizing and tracking machine learning experiments. It has a dedicated web app for visualizing training metrics in real-time. It also lets us manage production models centrally. We will use WandB only to track our Tiny-Llama fine-tuning run.

To use WandB, sign up for a free account and create an [API key](https://wandb.ai/authorize).

**Now, let’s start fine-tuning our model.**

How to Fine-tune Tiny-Llama?
----------------------------

Fine-tuning is a compute-heavy task. It requires a machine with 10-15 GB of VRAM, or you can use Colab’s free Tesla T4 GPU runtime.

Now install Unsloth and WandB

```python
%%capture
import torch
major_version, minor_version = torch.cuda.get_device_capability()
!pip install wandb
if major_version >= 8:
    # Use this for new GPUs like Ampere, Hopper GPUs (RTX 30xx, RTX 40xx, A100, H100, L40)
    !pip install "unsloth[colab_ampere] @ git+https://github.com/unslothai/unsloth.git"
else:
    # Use this for older GPUs (V100, Tesla T4, RTX 20xx)
    !pip install "unsloth[colab] @ git+https://github.com/unslothai/unsloth.git"
pass
```

The next thing is to load the 4-bit quantized pre-trained model with Unsloth.

```python
from unsloth import FastLanguageModel
import torch
max_seq_length = 4096 # Choose any! We auto support RoPE Scaling internally!
dtype = None # None for auto detection. Float16 for Tesla T4, V100, Bfloat16 for Ampere+
load_in_4bit = True # Use 4bit quantization to reduce memory usage. Can be False.

model, tokenizer = FastLanguageModel.from_pretrained(
    model_name = "unsloth/tinyllama-bnb-4bit", # "unsloth/tinyllama" for 16bit loading
    max_seq_length = max_seq_length,
    dtype = dtype,
    load_in_4bit = load_in_4bit,
)
```

This will install the model locally. The 4-bit model size will be around 760 MBs.

Now apply [PEFT](https://huggingface.co/blog/peft) to the 4-bit Tiny-Llama model.

```python
model = FastLanguageModel.get_peft_model(
    model,
    r = 32, # Choose any number > 0 ! Suggested 8, 16, 32, 64, 128
    target_modules = ["q_proj", "k_proj", "v_proj", "o_proj",
                      "gate_proj", "up_proj", "down_proj",],
    lora_alpha = 32,
    lora_dropout = 0, # Currently only supports dropout = 0
    bias = "none",    # Currently only supports bias = "none"
    use_gradient_checkpointing = True, # @@@ IF YOU GET OUT OF MEMORY - set to True @@@
    random_state = 3407,
    use_rslora = False,  # We support rank stabilized LoRA
    loftq_config = None, # And LoftQ
)
```

### Prepare Data

The next step is to prepare the dataset for fine-tuning. As I mentioned earlier, we will use a cleaned [Alpaca dataset](https://huggingface.co/datasets/yahma/alpaca-cleaned). This is a cleaned version of the original Alpaca dataset. It follows the instruction-input-response format. Here is an example of Alpaca data

![Image 3: Fine-Tuning](https://av-eks-lekhak.s3.amazonaws.com/media/__sized__/article_images/Screenshot_from_2024-01-30_18-12-22-thumbnail_webp-600x300.webp)

Now, let’s prepare our data.

```python
@title prepare data

#alpaca_prompt = """Below is an instruction that describes a task, paired with an input that
 provides further context.
 Write a response that appropriately completes the request.

### Instruction:
{}

### Input:
{}

### Response:
{}"""

EOS_TOKEN = tokenizer.eos_token
def formatting_prompts_func(examples):
    instructions = examples["instruction"]
    inputs       = examples["input"]
    outputs      = examples["output"]
    texts = []
    for instruction, input, output in zip(instructions, inputs, outputs):
        # Must add EOS_TOKEN, otherwise your generation will go on forever!
        text = alpaca_prompt.format(instruction, input, output) + EOS_TOKEN
        texts.append(text)
    return { "text" : texts, }
pass

from datasets import load_dataset
dataset = load_dataset("yahma/alpaca-cleaned", split = "train")
dataset = dataset.map(formatting_prompts_func, batched = True,)
```

Now, split the data into train and eval data. I have taken small eval data as larger eval data slows down the training.

```
dataset_dict = dataset.train_test_split(test_size=0.004)
```

### Configure WandB

Now, configure Weights and Biases in your current runtime.

```
# @title wandb init
import wandb
wandb.login()
```

Provide API key to log in to WandB when prompted.

Set up environment variables.

```
%env WANDB_WATCH=all
%env WANDB_SILENT=true
```

### Train Model

So far, we have loaded the 4-bit model, created the LoRA configuration, prepared the dataset, and configured WandB. The next step is to train the model on the data. For that, we need to define a trainer from the Trl library. We will use the SFTrainer from Trl. But before that, initialize WandB and define appropriate training arguments.

```
import os

from trl import SFTTrainer
from transformers import TrainingArguments
from transformers.utils import logging
import wandb

logging.set_verbosity_info()
project_name = "tiny-llama" 
entity = "wandb"
# os.environ["WANDB_LOG_MODEL"] = "checkpoint"

wandb.init(project=project_name, name = "tiny-llama-unsloth-sft")
```

Training Arguments

```
args = TrainingArguments(
        per_device_train_batch_size = 2,
        per_device_eval_batch_size=2,
        gradient_accumulation_steps = 4,
        evaluation_strategy="steps",
        warmup_ratio = 0.1,
        num_train_epochs = 1,
        learning_rate = 2e-5,
        fp16 = not torch.cuda.is_bf16_supported(),
        bf16 = torch.cuda.is_bf16_supported(),
        optim = "adamw_8bit",
        weight_decay = 0.1,
        lr_scheduler_type = "linear",
        seed = 3407,
        output_dir = "outputs",
        report_to="wandb",  # enable logging to W&B
        # run_name="tiny-llama-alpaca-run",  # name of the W&B run (optional)
        logging_steps=1,  # how often to log to W&B
        logging_strategy = 'steps',
        save_total_limit=2,
    )
```

This is important for training. To keep GPU usage low, keep the train, eval batch, and gradient accumulating steps low. The logging\_steps is the number of steps before metrics are logged to WandB.

Now, initialize the SFTrainer.

```
trainer = SFTTrainer(
    model = model,
    tokenizer = tokenizer,
    train_dataset = dataset_dict["train"],
    eval_dataset=dataset_dict["test"],
    dataset_text_field = "text",
    max_seq_length = max_seq_length,
    dataset_num_proc = 2,
    packing = True, # Packs short sequences together to save time!
    args = args,
)
```

Now, start the training.

```
trainer_stats = trainer.train()
wandb.finish()
```

During the training run, WandB will track the training and eval metrics. You visit the given dashboard link and see it in real-time.

This is a screenshot from my run on a Colab notebook.

![Image 4: Fine-tuning](https://av-eks-lekhak.s3.amazonaws.com/media/__sized__/article_images/Screenshot_from_2024-01-30_18-44-45-thumbnail_webp-600x300.webp)

The training speed will depend on multiple factors, including the training and eval data sizes, train and eval batch size, and the number of epochs. If you encounter GPU usage issues, try reducing batch and gradient accumulation step sizes. The train batch size = batch\_size\_per\_device \* gradient\_accumulation\_steps. And the number of optimization steps = total training data/batch size. You can play with the parameters and see which works better.

You can visualize the training and evaluation loss of your training on the WandB dashboard.

Train Loss

![Image 5: Train Loss](https://av-eks-lekhak.s3.amazonaws.com/media/__sized__/article_images/train-loss-thumbnail_webp-600x300.webp)

Eval Loss

![Image 6: Eval Loss](https://av-eks-lekhak.s3.amazonaws.com/media/__sized__/article_images/eval-loss-thumbnail_webp-600x300.webp)

### Inferencing

You can save the LoRA adapters locally or push them to the HuggingFace Repository.

```
model.save_pretrained("lora_model") # Local saving
# model.push_to_hub("your_name/lora_model", token = "...") # Online saving
```

You can also load the saved model from the disk and use it for inferencing.

```
if False:
    from unsloth import FastLanguageModel
    model, tokenizer = FastLanguageModel.from_pretrained(
        model_name = "lora_model", # YOUR MODEL YOU USED FOR TRAINING
        max_seq_length = max_seq_length,
        dtype = dtype,
        load_in_4bit = load_in_4bit,
    )

inputs = tokenizer(
[
    alpaca_prompt.format(
        "capital of France?", # instruction
        "", # input
        "", # output - leave this blank for a generation!
    )
]*1, return_tensors = "pt").to("cuda")

outputs = model.generate(**inputs, max_new_tokens = 64, use_cache = True)
tokenizer.batch_decode(outputs)
```

For streaming Model responses.

```
from transformers import TextStreamer

text_streamer = TextStreamer(tokenizer)
_ = model.generate(**inputs, streamer = text_streamer, max_new_tokens = 64)
```

So, this was all about fine-tuning a Tiny-Llama model with WandB logging.

Here is the [Colab Notebook](https://colab.research.google.com/drive/1RbTfG85-fHaUR8eSfUIxrFKK6bnDOVlM?usp=sharing) for the same.

Conclusion
----------

Small LLMs can be beneficial for deploying on compute-restricted hardware, such as personal computers, mobile phones, and other wearables, etc. Fine-tuning allows these models to perform better on downstream tasks. In this article, we learned how to Fine-tune a base language model on a dataset.

#### Key Takeaways

*   Fine-tuning is the process of making a pre-trained model adapt to a specific new task.
*   Tiny-Llama is an LLM with only 1.1 billion parameters and is trained on 3 trillion tokens.
*   There are different ways to Fine-tune LLMs, like LoRA and QLoRA.
*   Unsloth is an open-source platform that provides CUDA-optimized LLMs to speed up fine-tuning LLMs.
*   Weights and Biases (WandB) is a tool for tracking and storing ML experiments.

Frequently Asked Questions
--------------------------

****Q1. What is LLM fine-tuning?****

A. Fine-tuning, in the context of machine learning, especially deep learning, is a technique where you take a pre-trained model and adapt it to a new, specific task.

****Q2. Can I Fine-tune LLMs for free?****

A. It is possible to Fine-tune smaller LLMs for free on Colab over the Tesla T4 GPU with QLoRA.

****Q3. What are the benefits of Fine-tuning LLM?****

A. Fine-tuning vastly enhances LLM’s capability to perform downstream tasks, like role play, code generation, etc.

****Q4. What is Tiny-Llama?****

A. Tiny-Llama trained on 3 trillion tokens is an LLM with 1.1B parameters. The model adopts the original Llama-2 architecture.

****Q5. What is Unsloth used for?****

A. Unsloth is an open-source tool that provides faster and more efficient LLM fine-tuning by optimizing GPU kernels with Triton.

**The media shown in this article is not owned by Analytics Vidhya and is used at the Author’s discretion.**

