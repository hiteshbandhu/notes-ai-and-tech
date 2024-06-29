link : https://huyenchip.com/2023/04/11/llm-engineering.html


### 1. Fine Tuning vs Prompting

There are 3 main factors when considering prompting vs. finetuning: data availability, performance, and cost.

If you have only a few examples, prompting is quick and easy to get started. There's a limit to how many examples you can include in your prompt due to the maximum input token length.

The number of examples you need to finetune a model to your task, of course, depends on the task and the model. In my experience, however, you can expect a noticeable change in your model performance if you finetune on 100s examples. However, the result might not be much better than prompting.

In How Many Data Points is a Prompt Worth? (2021), Scao and Rush found that a prompt is worth approximately 100 examples (caveat: variance across tasks and models is high - see image below). The general trend is that as you increase the number of examples, finetuning will give better model performance than prompting. There's no limit to how many examples you can use to finetune a model.

|   |   |
|---|---|
||The benefit of finetuning is two folds:|
||1. You can get better model performance: can use more examples, examples becoming part of the model's internal knowledge.|
||2. You can reduce the cost of prediction. The more instruction you can bake into your model, the less instruction you have to put into your prompt. Say, if you can reduce ik tokens in your prompt for each prediction, for IM predictions on gpt-3.5-turbo, you'd save $2000.|


### 2.  What is Distillation ?

Finetuning with distillation

In March 2023, a group of Stanford students released a promising idea: finetune a smaller open-source language model (LLaMA-7B, the 7 billion parameter version of LLaMA) on examples generated by a larger language model (text-davinci-003 - 175 billion parameters). This technique of training a small model to imitate the behavior of a larger model is called distillation. The resulting finetuned model behaves similarly to text-davinci-003, while being a lot smaller and cheaper to run.

For finetuning, they used 52k instructions, which they inputted into text-davinci-003 to obtain outputs, which are then used to finetune LLaMa-7B. This costs under $500 to generate. The training process for finetuning costs under $100. See Stanford Alpaca: An Instruction-following LLaMA Model.

the model github : https://github.com/tatsu-lab/stanford_alpaca
paper to read : [https://arxiv.org/abs/2305.02301](https://arxiv.org/abs/2305.02301)

## Agents and Control Flows

![[Pasted%20image%20240630011647.png]]
