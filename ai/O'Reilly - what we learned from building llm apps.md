 
***link : https://www.oreilly.com/radar/what-we-learned-from-a-year-of-building-with-llms-part-i/***


***reference given to this article. : https://www.godaddy.com/resources/news/llm-from-the-trenches-10-lessons-learned-operationalizing-models-at-godaddy#h-1-sometimes-one-prompt-isn-t-enough

*reading for evals : https://hamel.dev/blog/posts/evals/#step-1-write-scoped-tests*



---

```
this article focuses on the methods to use while developing ai apps, and solutions to problems faced by reseachers at oreilly, they say they are from different fields, but have almost faced the same challenges building llm apps.
```

This work is organized into three sections: tactical, operational, and strategic.
### 1. Tactical

```
In this section, we share best practices for the core components of the emerging LLM stack: prompting tips to improve quality and reliability, evaluation strategies to assess output, retrieval-augmented generation ideas to improve grounding, and more. We also explore how to design human-in-the-loop workflows
```

#### Prompting

It’s underestimated because the right prompting techniques, when used correctly, can get us very far. It’s overestimated because even prompt-based applications require significant engineering around the prompt to work well.

A few prompting techniques have consistently helped improve performance across various models and tasks: n-shot prompts + in-context learning, chain-of-thought, and providing relevant resources.

##### N-shot prompting

The idea of in-context learning via n-shot prompts is to provide the LLM with a few examples that demonstrate the task and align outputs to our expectations. A few tips: 

- If n is too low, the model may over-anchor on those specific examples, hurting its ability to generalize. ``As a rule of thumb, aim for n ≥ 5. Don’t be afraid to go as high as a few dozen.``
- Examples should be representative of the expected input distribution. If you’re building a movie summarizer, include samples from different genres in roughly the proportion you expect to see in practice.
- You don’t necessarily need to provide the full input-output pairs. In many cases, examples of desired outputs are sufficient.
- If you are using an LLM that supports tool use, your n-shot examples should also use the tools you want the agent to use.

##### Chain of Though Prompting

For example, when asking an LLM to summarize a meeting transcript, we can be explicit about the steps, such as:

- First, list the key decisions, follow-up items, and associated owners in a sketchpad.
- Then, check that the details in the sketchpad are factually consistent with the transcript.
- Finally, synthesize the key points into a concise summary.

##### RAG Prompt Tips

When providing the relevant resources, it’s not enough to merely include them; don’t forget to tell the model to prioritize their use, refer to them directly, and sometimes to mention when none of the resources are sufficient. These help “ground” agent responses to a corpus of resources.

##### Use structured output

When using structured input, be aware that each LLM family has their own preferences. Claude prefers `xml` while GPT favors Markdown and JSON. With XML, you can even pre-fill Claude’s responses by providing a `response` tag like so.

> [!TODO] TODO : add link
>`claude : fill claude's mouth with response`

##### make small prompts that do one thing very well

Just like how we strive (read: struggle) to keep our systems and code simple, so should we for our prompts. Instead of having a single, catch-all prompt for the meeting transcript summarizer, we can break it into steps to:

- Extract key decisions, action items, and owners into structured format
- Check extracted details against the original transcription for consistency
- Generate a concise summary from the structured details

##### have good data, if using RAG - poor data, poor answers

#### Retrieval Augmented Generation

- RAG is only as good as the retrieved documents’ relevance, density, and detail
- The quality of your RAG’s output is dependent on the quality of retrieved documents, which in turn can be considered along a few factors

The first and most obvious metric is relevance. This is typically quantified via ranking metrics such as [Mean Reciprocal Rank (MRR)](https://en.wikipedia.org/wiki/Mean_reciprocal_rank) or [Normalized Discounted Cumulative Gain (NDCG)](https://en.wikipedia.org/wiki/Discounted_cumulative_gain). MRR evaluates how well a system places the first relevant result in a ranked list while NDCG considers the relevance of all the results and their positions. They measure how good the system is at ranking relevant documents higher and irrelevant documents lower. For example, if we’re retrieving user summaries to generate movie review summaries, we’ll want to rank reviews for the specific movie higher while excluding reviews for other movies.

>Like traditional recommendation systems, the rank of retrieved items will have a significant impact on how the LLM performs on downstream tasks. To measure the impact, run a RAG-based task but with the retrieved items shuffled—how does the RAG output perform?

>> Second, we also want to consider information density. If two documents are equally relevant, we should prefer one that’s more concise and has lesser extraneous details. Returning to our movie example, we might consider the movie transcript and all user reviews to be relevant in a broad sense. Nonetheless, the top-rated reviews and editorial reviews will likely be more dense in information.

>>>Finally, consider the level of detail provided in the document. Imagine we’re building a RAG system to generate SQL queries from natural language. We could simply provide table schemas with column names as context. But, what if we include column descriptions and some representative values? The additional detail could help the LLM better understand the semantics of the table and thus generate more correct SQL.

##### Use keyword-search, it can't beat similarity search

Second, it’s more straightforward to understand why a document was retrieved with keyword search—we can look at the keywords that match the query. In contrast, embedding-based retrieval is less interpretable. Finally, thanks to systems like Lucene and OpenSearch that have been optimized and battle-tested over decades, keyword search is usually more computationally efficient.

> In most cases, a hybrid will work best: keyword matching for the obvious matches, and embeddings for synonyms, hypernyms, and spelling errors, as well as multimodality (e.g., images and text). [Shortwave shared how they built their RAG pipeline](https://www.shortwave.com/blog/deep-dive-into-worlds-smartest-email-ai/), including query rewriting, keyword + embedding retrieval, and ranking.

>[!eye] look this up

##### Prefer RAG over fine-tuning for new knowledge

Beyond improved performance, RAG comes with several practical advantages too. First, compared to continuous pretraining or fine-tuning, it’s easier—and cheaper!—to keep retrieval indices up-to-date. Second, if our retrieval indices have problematic documents that contain toxic or biased content, we can easily drop or modify the offending documents.

In addition, the R in RAG provides finer grained control over how we retrieve documents

##### Long-context models won’t make RAG obsolete

First, even with a context window of 10M tokens, we’d still need a way to select information to feed into the model. Second, beyond the narrow needle-in-a-haystack eval, we’ve yet to see convincing data that models can effectively reason over such a large context. Thus, without good retrieval (and ranking), we risk overwhelming the model with distractors, or may even fill the context window with completely irrelevant information.

>Finally, there’s cost. `The Transformer’s inference cost scales quadratically` (or linearly in both space and time) with context length. Just because there exists a model that could read your organization’s entire Google Drive contents before answering each question doesn’t mean that’s a good idea. Consider an analogy to how we use RAM: we still read and write from disk, even though there exist compute instances with [RAM running into the tens of terabytes](https://aws.amazon.com/ec2/instance-types/high-memory/).

####  Tuning and optimizing workflows

Some things to try

- An explicit planning step, as tightly specified as possible. Consider having predefined plans to choose from (c.f. https://youtu.be/hGXhFa3gzBs?si=gNEGYzux6TuB1del).
- Rewriting the original user prompts into agent prompts. Be careful, this process is lossy!
- Agent behaviors as linear chains, DAGs, and State-Machines; different dependency and logic relationships can be more and less appropriate for different scales. Can you squeeze performance optimization out of different task architectures?
- Planning validations; your planning can include instructions on how to evaluate the responses from other agents to make sure the final assembly works well together.
- Prompt engineering with fixed upstream state—make sure your agent prompts are evaluated against a collection of variants of what may happen before.

#### Prioritize deterministic workflows for now

While AI agents can dynamically react to user requests and the environment, their non-deterministic nature makes them a challenge to deploy. Each step an agent takes has a chance of failing, and the chances of recovering from the error are poor.

Make plans. This allows each step to be more predictable and reliable. Benefits include:

- Generated plans can serve as few-shot samples to prompt or finetune an agent.
- Deterministic execution makes the system more reliable, and thus easier to test and debug. Furthermore, failures can be traced to the specific steps in the plan.
- Generated plans can be represented as directed acyclic graphs (DAGs) which are easier, relative to a static prompt, to understand and adapt to new situations.


#### Don't rely on temperature only 

increasing temperature does not guarantee that the LLM will sample outputs from the probability distribution you expect (e.g., uniform random). Nonetheless, we have other tricks to increase output diversity. 

The simplest way is to adjust elements within the prompt. For example, if the prompt template includes a list of items, such as historical purchases, shuffling the order of these items each time they’re inserted into the prompt can make a significant difference. 

Additionally, keeping a short list of recent outputs can help prevent redundancy. In our recommended products example, by instructing the LLM to avoid suggesting items from this recent list, or by rejecting and resampling outputs that are similar to recent suggestions, we can further diversify the responses. Another effective strategy is to vary the phrasing used in the prompts. For instance, incorporating phrases like “pick an item that the user would love using regularly” or “select a product that the user would likely recommend to friends” can shift the focus and thereby influence the variety of recommended products.

#### Caching is underrated.

Caching saves cost and eliminates generation latency by removing the need to recompute responses for the same input. Furthermore, if a response has previously been guardrailed, we can serve these vetted responses and reduce the risk of serving harmful or inappropriate content.

One straightforward approach to caching is to use unique IDs for the items being processed, such as if we’re summarizing new articles or [product reviews](https://www.cnbc.com/2023/06/12/amazon-is-using-generative-ai-to-summarize-product-reviews.html). When a request comes in, we can check to see if a summary already exists in the cache. If so, we can return it immediately; if not, we generate, guardrail, and serve it, and then store it in the cache for future requests.

>For more open-ended queries, we can borrow techniques from the field of search, which also leverages caching for open-ended inputs. Features like autocomplete and spelling correction also help normalize user input and thus increase the cache hit rate.

#### When to fine-tune

We may have some tasks where even the most cleverly designed prompts fall short. For example, even after significant prompt engineering, our system may still be a ways from returning reliable, high-quality output. If so, then it may be necessary to finetune a model for your specific task.

If prompting gets you 90% of the way there, then fine-tuning may not be worth the investment

#### Evaluation & Monitoring

##### Create a few assertion-based unit tests from real input/output samples

Create [unit tests (i.e., assertions)](https://hamel.dev/blog/posts/evals/#level-1-unit-tests) consisting of samples of inputs and outputs from production, with expectations for outputs based on at least three criteria. While three criteria might seem arbitrary, it’s a practical number to start with; fewer might indicate that your task isn’t sufficiently defined or is too open-ended, like a general-purpose chatbot. These unit tests, or assertions, should be triggered by any changes to the pipeline, whether it’s editing a prompt, adding new context via RAG, or other modifications. This [write-up has an example](https://hamel.dev/blog/posts/evals/#step-1-write-scoped-tests) of an assertion-based test for an actual use case.


#### LLM as a judge technique of evaluation



