
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



## Pre-fill Claude's response for greater output control

When using Claude, you have the unique ability to guide its responses by prefilling the `Assistant` message. This powerful technique allows you to direct Claude’s actions, skip preambles, enforce specific formats like JSON or XML, and even help Claude maintain character consistency in role-play scenarios.

```python
import anthropic

client = anthropic.Anthropic()
response = client.messages.create(
    model="claude-3-5-sonnet-20240620",
    max_tokens=1024,
    messages=[
        {"role": "user", "content": "What is your favorite color?"},
        {"role": "assistant", "content": "As an AI assistant, I don't have a favorite color, But if I had to pick, it would be green because"}  # Prefill here
    ]
)
```

> **Power user tip**: Prefilling `{` forces Claude to skip the preamble and directly output the JSON object. This is cleaner, more concise, and easier for programs to parse without additional processing.

**example** : 

|Role|Content|
|---|---|
|User|Extract the name, size, price, and color from this product description as a JSON object:  <br>  <br><description>  <br>The SmartHome Mini is a compact smart home assistant available in black or white for only $49.99. At just 5 inches wide, it lets you control lights, thermostats, and other connected devices via voice or app—no matter where you place it in your home. This affordable little hub brings convenient hands-free control to your smart devices.  <br></description>|
|Assistant (prefill)|{|
|Assistant (Claude’s response)|“name”: “SmartHome Mini”,  <br>“size”: “5 inches wide”,  <br>“price”: “$49.99”,  <br>“colors”: [  <br>“black”,  <br>“white”  <br>]  <br>}|
##### Example 2: Maintaining character in roleplay scenarios

|Role|Content|
|---|---|
|User (50 conversation turns in with “Sherlock Holmes” Claude)|What do you deduce about the owner of this shoe?  <br><image>muddy_shoe.jpg</image>|
|Assistant (prefill)|[Sherlock Holmes]|
|Assistant (Claude’s response)|Ah, what have we here? A most curious specimen, Watson! Let us examine this shoe with a discerning eye. The mud, you see, is not your common London clay, but rather the distinctive reddish hue of the iron-rich soil found only in the environs of Hampstead Heath. And yet, the wear on the sole suggests a gait more suited to the cobblestones of Whitechapel than the gentle slopes of the Heath.  <br>  <br>Now, observe the laces! They are tied with a particular knot favored by those in the medical profession—doubtless a habit acquired during years of tying surgical gowns. But the scuff marks on the toe box tell a different tale: they are consistent with the kicking motion one might employ in the martial arts, specifically Baritsu.  <br>  <br>We may conclude, then, that our absent shoe owner is a doctor, possibly a surgeon, who resides in Whitechapel but frequents Hampstead Heath—perhaps for a secret Baritsu practice, away from prying eyes. A most intriguing case, Watson! We must make haste; the game is afoot!|
#### Chain prompts so that each step get claude's attention

#### Prompt 1

|Role|Content|
|---|---|
|User|You’re our Chief Legal Officer. Review this SaaS contract for risks, focusing on data privacy, SLAs, and liability caps.  <br>  <br><contract>  <br>{{CONTRACT}}  <br></contract>  <br>  <br>Output your findings in <risks> tags.|
|Assistant  <br>{{CONCERNS}}|1. Data Privacy (Section 7):  <br>- Issue: Vendor has broad rights to use client data for “service improvement.”  <br>- Risk: Could lead to unauthorized data sharing or monetization of client data.  <br>- Impact: Potential breach of client trust, regulatory fines (GDPR, CCPA).  <br>  <br>2. SLAs (Section 5):  <br>- Issue: 99.5% uptime guarantee is below industry standard (typically 99.99%).  <br>- Risk: Up to 43.8 hours of downtime per year vs. 52.6 minutes at 99.99%.  <br>- Impact: Operational disruptions, revenue loss, client dissatisfaction.  <br>  <br>3. Liability (Section 9):  <br>- Issue: Liability cap set at 3 months of fees.  <br>- Risk: Inadequate for potential data breaches, IP theft, or prolonged service failures.  <br>- Impact: Insufficient compensation for damages that could far exceed the cap, like class-action lawsuits or regulatory fines.|

#### Prompt 2

|Role|Content|
|---|---|
|User|Draft an email to a SaaS product vendor outlining the following concerns and proposing changes. Here are the concerns:  <br><concerns>  <br>{{CONCERNS}}  <br></concerns>|
|Assistant  <br>{{EMAIL}}|Dear {{VENDOR}},  <br>  <br>After a thorough review of the proposed SaaS agreement, our legal team has identified several high-priority concerns that must be addressed before we can proceed:  <br>  <br>1. Data Privacy (Section 7):  <br>- Current: Broad rights to use client data for “service improvement.”  <br>- Concern: Potential unauthorized data sharing or monetization.  <br>- Proposed Change: Limit data use to direct service provision. Any use for improvement, analytics, or marketing must be opt-in, revocable, and anonymized.  <br>  <br>2. SLAs (Section 5):  <br>- Current: 99.5% uptime guarantee.  <br>- Concern: Up to 43.8 hours of downtime per year, risking operational disruptions.  <br>- Proposed Change: 99.99% uptime with tiered service credits. E.g., 10% credit for <99.99%, 25% for <99.9%, 50% for <99.5%.  <br>  <br>3. Liability (Section 9):  <br>- Current: Cap at 3 months of fees.  <br>- Concern: Grossly inadequate for data breaches, IP theft, or prolonged outages.  <br>- Proposed Change: 12 months’ fees for general liability. Uncapped for data breaches, willful misconduct, IP infringement.  <br>  <br>These changes are non-negotiable for us. They align with industry standards and our commitment to data security, operational excellence, and fiduciary responsibility. We value the potential of our partnership, and addressing these will set a strong foundation.  <br>  <br>Please have your legal team review and incorporate these changes. We’re happy to discuss further to ensure mutual understanding and agreement.  <br>  <br>Best regards,  <br>{{CLO_NAME}}|

#### Prompt 3

| Role      | Content                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                               |
| --------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| User      | Your task is to review an email and provide feedback. Here is the email:  <br><email>  <br>{{EMAIL}}  <br></email>  <br>  <br>Give feedback on tone, clarity, and professionalism.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                    |
| Assistant | 1. Tone:  <br>- Professional and assertive without being confrontational.  <br>- Use of “non-negotiable” is strong but justified given the criticality of the issues.  <br>- Collaborative language (“we value,” “happy to discuss”) balances the firmness, maintaining a constructive tone.  <br>  <br>2. Clarity:  <br>- Excellent structuring: Current state, concern, and proposed change for each issue.  <br>- Technical terms (e.g., “EBITDA,” “tiered service credits”) are appropriate for a B2B SaaS context.  <br>- Action items are clear: vendor’s legal team to review and incorporate changes.  <br>  <br>3. Professionalism:  <br>- Focuses on mutual benefit and industry standards, not just our demands.  <br>- Mentions broader responsibilities (data security, operational excellence, fiduciary duty), elevating the discussion.  <br>- Offers further discussion, showing commitment to understanding and resolving issues.  <br>  <br>Overall, the email strikes the right balance of assertiveness and collaboration. It clearly articulates our concerns, proposes specific solutions, and maintains a professional, partnership-oriented tone. No revisions needed; proceed with sending. |


#### Tips for Long Context in Claude

**Put longform data at the top**: Place your long documents and inputs (~20K+ tokens) near the top of your prompt, above your query, instructions, and examples. This can significantly improve Claude’s performance across all models.

> Queries at the end can improve response quality by up to 30% in tests, especially with complex, multi-document inputs.


**Structure document content and metadata with XML tags**: When using multiple documents, wrap each document in `<document>` tags with `<document_content>` and `<source>` (and other metadata) subtags for clarity.

```xml
<documents>
  <document index="1">
    <source>annual_report_2023.pdf</source>
    <document_content>
      {{ANNUAL_REPORT}}
    </document_content>
  </document>
  <document index="2">
    <source>competitor_analysis_q2.xlsx</source>
    <document_content>
      {{COMPETITOR_ANALYSIS}}
    </document_content>
  </document>
</documents>

Analyze the annual report and competitor analysis. Identify strategic adva
```

**round responses in quotes**: For long document tasks, ask Claude to quote relevant parts of the documents first before carrying out its task. This helps Claude cut through the “noise” of the rest of the document’s contents.

```xml
You are an AI physician's assistant. Your task is to help doctors diagnose possible patient illnesses.

<documents>
  <document index="1">
    <source>patient_symptoms.txt</source>
    <document_content>
      {{PATIENT_SYMPTOMS}}
    </document_content>
  </document>
  <document index="2">
    <source>patient_records.txt</source>
    <document_content>
      {{PATIENT_RECORDS}}
    </document_content>
  </document>
  <document index="3">
    <source>patient01_appt_history.txt</source>
    <document_content>
      {{PATIENT01_APPOINTMENT_HISTORY}}
    </document_content>
  </document>
</documents>

Find quotes from the patient records and appointment history that are rele
```



### how to place images

> Just as with document-query placement, Claude works best when images come before text. Images placed after text or interpolated with text will still perform well, but if your use case allows it, we recommend an image-then-text structure.

