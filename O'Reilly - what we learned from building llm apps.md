
***link : https://www.oreilly.com/radar/what-we-learned-from-a-year-of-building-with-llms-part-i/***


***reference given to this article. : https://www.godaddy.com/resources/news/llm-from-the-trenches-10-lessons-learned-operationalizing-models-at-godaddy#h-1-sometimes-one-prompt-isn-t-enough

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

- If n is too low, the model may over-anchor on those specific examples, hurting its ability to generalize. As a rule of thumb, aim for n ≥ 5. Don’t be afraid to go as high as a few dozen.
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

Like traditional recommendation systems, the rank of retrieved items will have a significant impact on how the LLM performs on downstream tasks. To measure the impact, run a RAG-based task but with the retrieved items shuffled—how does the RAG output perform?

Second, we also want to consider information density. If two documents are equally relevant, we should prefer one that’s more concise and has lesser extraneous details. Returning to our movie example, we might consider the movie transcript and all user reviews to be relevant in a broad sense. Nonetheless, the top-rated reviews and editorial reviews will likely be more dense in information.

Finally, consider the level of detail provided in the document. Imagine we’re building a RAG system to generate SQL queries from natural language. We could simply provide table schemas with column names as context. But, what if we include column descriptions and some representative values? The additional detail could help the LLM better understand the semantics of the table and thus generate more correct SQL.

##### Use keyword-search, it can't beat similarity search

