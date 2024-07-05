
***link : https://www.oreilly.com/radar/what-we-learned-from-a-year-of-building-with-llms-part-i/***

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

> [!TODO] TODO : add li
>`claude : fill claude's mouth with response`





