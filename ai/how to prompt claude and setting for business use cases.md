
link : https://docs.anthropic.com/en/docs/build-with-claude/prompt-engineering/overview#prompting-vs-finetuning\

link : https://www.anthropic.com/news/prompt-engineering-for-business-performance

---


### prompting vs fine-tuning

Prompt engineering is far faster than other methods of model behavior control, such as finetuning, and can often yield leaps in performance in far less time. Here are some reasons to consider prompt engineering over finetuning:  

- **Resource efficiency**: Fine-tuning requires high-end GPUs and large memory, while prompt engineering only needs text input, making it much more resource-friendly.
- **Cost-effectiveness**: For cloud-based AI services, fine-tuning incurs significant costs. Prompt engineering uses the base model, which is typically cheaper.
- **Maintaining model updates**: When providers update models, fine-tuned versions might need retraining. Prompts usually work across versions without changes.
- **Time-saving**: Fine-tuning can take hours or even days. In contrast, prompt engineering provides nearly instantaneous results, allowing for quick problem-solving.
- **Minimal data needs**: Fine-tuning needs substantial task-specific, labeled data, which can be scarce or expensive. Prompt engineering works with few-shot or even zero-shot learning.
- **Flexibility & rapid iteration**: Quickly try various approaches, tweak prompts, and see immediate results. This rapid experimentation is difficult with fine-tuning.
- **Domain adaptation**: Easily adapt models to new domains by providing domain-specific context in prompts, without retraining.
- **Comprehension improvements**: Prompt engineering is far more effective than finetuning at helping models better understand and utilize external content such as retrieved documents
- **Preserves general knowledge**: Fine-tuning risks catastrophic forgetting, where the model loses general knowledge. Prompt engineering maintains the model’s broad capabilities.
- **Transparency**: Prompts are human-readable, showing exactly what information the model receives. This transparency aids in understanding and debugging.


---
## prompting techniques for models


1. Be clear, and show it to colleague with minimal context of the problem. if the new colleague is confused in reading the prompt and following the instructions, the model will be too
#### How to be clear, contextual, and specific

- **Give Claude contextual information:** Just like you might be able to better perform on a task if you knew more context, Claude will perform better if it has more contextual information. Some examples of contextual information:
    - What the task results will be used for
    - What audience the output is meant for
    - What workflow the task is a part of, and where this task belongs in that workflow
    - The end goal of the task, or what a successful task completion looks like
- **Be specific about what you want Claude to do:** For example, if you want Claude to output only code and nothing else, say so.

> **Provide instructions as sequential steps:** Use numbered lists or bullet points to better ensure that Claude carries out the task the exact way you want it to.


#### GOOD VS BAD PROMPTS

|      | bad prompt                                              | good prompt                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                  |
| ---- | ------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| User | Write a marketing email for our new AcmeCloud features. | Your task is to craft a targeted marketing email for our Q3 AcmeCloud feature release.  <br>  <br>Instructions:  <br>1. Write for this target audience: Mid-size tech companies (100-500 employees) upgrading from on-prem to cloud.  <br>2. Highlight 3 key new features: advanced data encryption, cross-platform sync, and real-time collaboration.  <br>3. Tone: Professional yet approachable. Emphasize security, efficiency, and teamwork.  <br>4. Include a clear CTA: Free 30-day trial with priority onboarding.  <br>5. Subject line: Under 50 chars, mention “security” and “collaboration”.  <br>6. Personalization: Use {{COMPANY_NAME}} and {{CONTACT_NAME}} variables.  <br>  <br>Structure:  <br>1. Subject line  <br>2. Email body (150-200 words)  <br>3. CTA button text |

### Use examples to steer your output

1. This technique, known as few-shot or multishot prompting, is particularly effective for tasks that require structured outputs or adherence to specific formats.
2. For maximum effectiveness, make sure that your examples are:

	- **Relevant**: Your examples mirror your actual use case.
	- **Diverse**: Your examples cover edge cases and potential challenges, and vary enough that Claude doesn’t inadvertently pick up on unintended patterns.
	- **Clear**: Your examples are wrapped in `<example>` tags (if multiple, nested within `<examples>` tags) for structure.

> Ask Claude to evaluate your examples for relevance, diversity, or clarity. Or have Claude generate more examples based on your initial set.


### Use chain of though prompting

When faced with complex tasks like research, analysis, or problem-solving, giving Claude space to think can dramatically improve its performance. This technique, known as chain of thought (CoT) prompting, encourages Claude to break down problems step-by-step, leading to more accurate and nuanced outputs.


**Structured prompt**: Use XML tags like `<thinking>` and `<answer>` to separate reasoning from the final answer.

| Role | Content                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                    |
| ---- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| User | Draft personalized emails to donors asking for contributions to this year’s Care for Kids program.  <br>  <br>Program information:  <br><program>{{PROGRAM_DETAILS}}  <br></program>  <br>  <br>Donor information:  <br><donor>{{DONOR_DETAILS}}  <br></donor>  <br>  <br>Think before you write the email in \<thinking> tags. First, think through what messaging might appeal to this donor given their donation history and which campaigns they’ve supported in the past. Then, think through what aspects of the Care for Kids program would appeal to them, given their history. Finally, write the personalized donor email in \<email> tags, using your analysis. |


###  Use XML tags to structure your prompts

- this is the best trick in the bag for claude, as it is potentially trained on xml formatted data

> **XML tip**: Use tags like `<instructions>`, `<example>`, and `<formatting>` to clearly separate different parts of your prompt. This prevents Claude from mixing up instructions with examples or context.

---
### Give claude roles in the system prompt

```python
import anthropic

client = anthropic.Anthropic()

response = client.messages.create(
    model="claude-3-5-sonnet-20240620",
    max_tokens=2048,
    system="You are a seasoned data scientist at a Fortune 500 company.", # <-- role prompt
    messages=[
        {"role": "user", "content": "Analyze this dataset for anomalies: <dataset>{{DATASET}}</dataset>"}
    ]
)

print(response.content)
```
