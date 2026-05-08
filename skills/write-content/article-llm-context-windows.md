# What is an LLM context window? A developer's guide to limits, costs, and workarounds

An AI context window is the total number of tokens an LLM can process in a single request. Your prompt, conversation history, uploaded documents, system instructions, and the model's response all fight for the same fixed space. Think of it as the model's working memory. Everything outside it is invisible. Current windows range from 128K tokens on entry-level models to over 1 million on Gemini, Claude Opus, and GPT-5.5.

I hit this limit the hard way. Three hours into a coding session with Claude, I noticed the code it was generating didn't match the architecture I'd specified at the start. Variables renamed. Patterns swapped. The model wasn't broken. It had silently dropped my original instructions from its context window. Three hours of context had pushed those early messages out. `/compact` brought them back. The bill for that session was $8 in API costs, about $3 of which was reprocessing instructions the model had already forgotten.

Context windows depend entirely on [how LLMs process tokens](https://aioutlooks.com/what-are-ai-tokens/). If you're shaky on what a token actually is, read that first. This guide picks up where that one leaves off: how windows work, what they cost you, and how to build around the limits.

## Tokens, words, and pages: how much actually fits

Context windows are measured in tokens, not words. And tokens don't map cleanly to words. A token can be a whole word (`cat`), part of a word (`ness`), a single character (`a`), or punctuation (`.`). The conversion varies by model and language.

Here's the rough math for English prose:

<table>
  <thead>
    <tr>
      <th>Context window</th>
      <th>Approximate words</th>
      <th>Pages (single-spaced)</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>128K (most models)</td>
      <td>~96,000</td>
      <td>~200</td>
    </tr>
    <tr>
      <td>200K (Claude Haiku)</td>
      <td>~150,000</td>
      <td>~300</td>
    </tr>
    <tr>
      <td>400K (GPT-5.4, GPT-5.5)</td>
      <td>~300,000</td>
      <td>~600</td>
    </tr>
    <tr>
      <td>1M (Gemini, Claude Opus, Llama, DeepSeek)</td>
      <td>~750,000</td>
      <td>~1,500</td>
    </tr>
  </tbody>
</table>

The 0.75 words-per-token ratio breaks for code and structured data. A 200-character JSON object can eat 50-80 tokens because every brace, bracket, colon, and indentation gets its own token. The same 200 characters of natural prose might be 45-55 tokens. If you're sending structured API responses to a model, you're burning more context than you think.

Want to check your actual token count? Drop this into Python:

```python
import tiktoken

encoding = tiktoken.encoding_for_model("gpt-4o")
text = "Your prompt or document goes here"
tokens = encoding.encode(text)
print(f"Token count: {len(tokens)}")
# For rough word estimate in English:
print(f"Approximate words: {len(tokens) * 0.75:.0f}")
```

For Anthropic's models, use their API token counting endpoint. For Gemini, call `count_tokens()` before sending. Each provider has its own tokenizer. The same text will produce different counts across models, sometimes by 10-15%. Test with your actual data, not estimates.

## Why context windows exist (and can't just be "made bigger")

[Transformer models](https://aioutlooks.com/what-is-a-large-language-model/) use a self-attention mechanism that compares every token against every other token in the input. The computational cost scales quadratically: double the context, quadruple the work. A 10K-token input needs about 100 million comparisons. A 100K-token input needs 10 billion.

Then there's the KV cache. Every time the model generates a token, it stores key-value pairs for every previous token in GPU memory. As a conversation grows, so does this cache. Eventually you run out of VRAM. That's why chatbots get noticeably slower the longer you talk. The model is hauling around an ever-growing KV cache.

Modern models use tricks to push these limits: FlashAttention restructures memory access patterns for 2-4x speedups. Sparse attention skips distant token pairs that probably don't matter. Ring attention distributes long sequences across multiple GPUs. These work, but they don't change the fundamental relationship. More context still means more compute, more memory, and more money.

The practical upshot: a 1M-token model doesn't process 1M tokens as efficiently as a 128K model processes 128K. The top end of the window is always slower and more expensive per token than the sweet spot.

## Context windows across major models (May 2026)

<table>
  <thead>
    <tr>
      <th>Model</th>
      <th>Provider</th>
      <th>Context window</th>
      <th>Max output</th>
      <th>Input cost per 1M tokens</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>GPT-5.5</td>
      <td>OpenAI</td>
      <td>400,000+</td>
      <td>128,000</td>
      <td>$5.00</td>
    </tr>
    <tr>
      <td>GPT-5.4</td>
      <td>OpenAI</td>
      <td>400,000+</td>
      <td>128,000</td>
      <td>$2.50</td>
    </tr>
    <tr>
      <td>GPT-5.4-mini</td>
      <td>OpenAI</td>
      <td>128,000+</td>
      <td>64,000</td>
      <td>$0.75</td>
    </tr>
    <tr>
      <td>Claude Opus 4.7</td>
      <td>Anthropic</td>
      <td>1,000,000</td>
      <td>128,000</td>
      <td>$5.00</td>
    </tr>
    <tr>
      <td>Claude Sonnet 4.6</td>
      <td>Anthropic</td>
      <td>1,000,000</td>
      <td>64,000</td>
      <td>$3.00</td>
    </tr>
    <tr>
      <td>Claude Haiku 4.5</td>
      <td>Anthropic</td>
      <td>200,000</td>
      <td>64,000</td>
      <td>$1.00</td>
    </tr>
    <tr>
      <td>Gemini 2.5 Pro</td>
      <td>Google</td>
      <td>1,000,000</td>
      <td>65,536</td>
      <td>$1.25</td>
    </tr>
    <tr>
      <td>Gemini 2.5 Flash</td>
      <td>Google</td>
      <td>1,000,000</td>
      <td>65,536</td>
      <td>$0.15</td>
    </tr>
    <tr>
      <td>DeepSeek V4-Flash</td>
      <td>DeepSeek</td>
      <td>1,000,000</td>
      <td>384,000</td>
      <td>$0.14</td>
    </tr>
    <tr>
      <td>Llama 4 Maverick</td>
      <td>Meta</td>
      <td>1,000,000</td>
      <td>16,384</td>
      <td>Open-source</td>
    </tr>
  </tbody>
</table>

Two things stand out. First, Google gives you 1M tokens on both their cheap and expensive models, meaning context window size does not correlate with model capability. Second, max output is always dramatically smaller than max input. Claude Opus can read 1M tokens but can only write 128K. If you need the model to produce a book chapter, verify the output limit, not just the context window.

## How context windows work in practice

### Conversation memory: the silent drop

Every message you send and every response the model generates accumulates inside the context window. When the window fills up, the oldest tokens get dropped. No error message, no warning, no "hey, I forgot your first three instructions."

The model just starts behaving differently.

I learned this debugging a client's chatbot that was supposed to respond exclusively in Spanish. It worked fine for eight turns. By turn eleven, answers were coming back in English. The system instruction (`Always respond in Spanish`) had been pushed out of the context window. The model wasn't broken. The context was.

This silent dropping is the most dangerous thing about context windows. Your carefully written system prompt from message one? Gone. The instruction to never expose certain data? Vanished. The architecture pattern you specified three hours ago? Irrelevant. Nothing tells you this happened. You just notice the output quality drifting.

Most chat interfaces handle this by quietly trimming the oldest messages. API calls typically return an error when input exceeds the limit, but not always. Some platforms silently truncate. Always monitor your input token count per request.

### Document processing: when your PDF won't fit

A model with a 200K-token window can handle roughly 150K words of prose, or about 300 pages single-spaced. That sounds like a lot until you hit a 500-page legal contract or a codebase with 200 source files.

The limit is shared between input and output. Send 190K tokens of input to Claude Sonnet and you have 10K tokens left for the response. If your prompt asks for a detailed analysis, the model will run out of space before finishing. You get a truncated answer and no explanation.

For multimodal models, media counts against the window too. A single high-resolution image can consume roughly 1,032 tokens on Gemini. One second of video runs about 263 tokens. Upload a 30-second clip and you've burned nearly 8,000 tokens before asking a single question.

### The lost-in-the-middle problem

Even when text fits in the context window, the model doesn't give equal attention to every part. A 2023 paper by Liu et al. demonstrated that LLMs use information most reliably from the beginning and the end of long inputs. Information buried in the middle gets partially ignored.

This isn't a bug in a specific model. It's a property of how attention mechanisms distribute focus across long sequences. It's related to the broader question of whether LLMs truly reason or just pattern-match — a topic explored in depth in the article on [the illusion of AI reasoning](https://aioutlooks.com/the-illusion-of-ai-reasoning/). Newer models handle it better. Claude Opus 4.7 and GPT-5.5 show improved mid-context retrieval, but the effect hasn't disappeared.

What this means in practice: if you're building a RAG pipeline that retrieves 20 document chunks and stuffs them into the prompt, the most critical chunk had better land in the first or last few positions. Chunks 8 through 13, right in the middle, might as well be invisible. Put your most important information at the beginning, right after the system prompt. Put instructions and questions at the end, closest to where the model generates its response.

### Reasoning tokens: the invisible context eater

GPT-5.5, Claude with extended thinking, and Gemini with reasoning all generate hidden tokens you never see. The model works through a problem internally, producing "thinking tokens" that occupy context window space and get billed as output. A single complex query might generate 10,000 reasoning tokens before producing a 200-token final answer.

You pay for all 10,200 tokens. And those 10,000 thinking tokens consume context window space, reducing what's available for conversation history and future responses. This matters in long sessions. Reasoning models will hit the context limit faster than standard models because they're generating invisible content.

When to use reasoning models: complex multi-step problems where the extra thinking produces measurably different output. When to skip them: classification, extraction, summarization, translation, and simple Q&A. The reasoning cost isn't worth it when a standard model gets the same answer.

## The cost of filling the window

Context windows and API bills are the same thing from different angles. Every token in the window costs money.

Here's what it costs to fill each model's full context window with input tokens alone:

<table>
  <thead>
    <tr>
      <th>Model</th>
      <th>Context</th>
      <th>Cost to fill full context (input only)</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>GPT-5.5</td>
      <td>~400K</td>
      <td>$4.00 (long context rate)</td>
    </tr>
    <tr>
      <td>GPT-5.4-mini</td>
      <td>128K+</td>
      <td>$0.10</td>
    </tr>
    <tr>
      <td>Claude Opus 4.7</td>
      <td>1M</td>
      <td>$5.00</td>
    </tr>
    <tr>
      <td>Claude Sonnet 4.6</td>
      <td>1M</td>
      <td>$3.00</td>
    </tr>
    <tr>
      <td>Gemini 2.5 Flash</td>
      <td>1M</td>
      <td>$0.15</td>
    </tr>
    <tr>
      <td>DeepSeek V4-Flash</td>
      <td>1M</td>
      <td>$0.14</td>
    </tr>
  </tbody>
</table>

Gemini 2.5 Flash costs $0.15 to fill a 1M-token window. Claude Opus costs $5.00 for the same amount of context. Same amount of information, 33x price difference. The model, not the context size, drives the cost.

If your application sends the full conversation history with every turn, costs compound fast. A chatbot handling 10,000 conversations per day, each averaging 2,000 tokens of input and 500 tokens of output on GPT-5.4-mini, runs roughly $180 per month in API costs. The output tokens often cost as much as the input. Always calculate both sides.

Prompt caching can cut this dramatically. Claude charges $5.00 per million input tokens at standard rates but $0.50 per million for cached reads, a 90% discount on the part of your prompt that never changes. If you're sending the same system prompt and document set across multiple requests, cache it. This is one of several optimization tactics. [Stopping token waste in Claude Code](https://aioutlooks.com/how-to-stop-wasting-tokens/) covers the fixes that move the needle fastest.

System prompts themselves are a fixed cost on every call. A 2,000-token system prompt processed 10,000 times per day on Claude Sonnet 4.6 costs $60 per day, $1,800 per month, just for the system prompt. Keep system prompts short. Everything you put in them, you pay for on every single request.

## 5 strategies for working within (and around) context limits

### Strategy 1: Summarization chains (map-reduce)

When your document is too long to fit in one pass, split it, summarize each chunk independently, then combine the summaries for a final synthesis.

```python
def summarize_long_document(document, chunk_size=4000):
    chunks = [document[i:i+chunk_size] for i in range(0, len(document), chunk_size)]
    summaries = []

    for chunk in chunks:
        summary = call_llm(f"Summarize this section in 200 words:\n\n{chunk}")
        summaries.append(summary)

    combined = "\n\n".join(summaries)
    final = call_llm(f"Synthesize these section summaries into a coherent overview:\n\n{combined}")
    return final
```

This costs more in API calls (you're making N summaries plus one synthesis) but lets you process documents of unlimited length. Works best for summarization and extraction tasks where each chunk is mostly independent.

### Strategy 2: Chunking with RAG

Retrieval-Augmented Generation keeps your context small by storing documents externally and retrieving only the relevant chunks at query time. Instead of stuffing a 500-page manual into the context window, you store it in a vector database and pull the 5-10 most relevant passages. For a full walkthrough, see [building a multimodal RAG pipeline with Gemini File Search](https://aioutlooks.com/build-multimodal-rag-with-google-file-search/).

Use RAG when your data is larger than the context window, your queries are specific, and accuracy matters more than cross-document reasoning. Use full-context stuffing when the task requires reasoning across the entire dataset at once, comparing clauses across a contract, finding contradictions in a report, or analyzing a complete codebase.

```python
# Pseudocode for a RAG pipeline
query = "What are the cancellation terms?"

# 1. Embed the query
query_embedding = embed(query)

# 2. Retrieve top-5 most similar document chunks
similar_chunks = vector_db.search(query_embedding, top_k=5)

# 3. Stuff retrieved chunks + query into the context window
prompt = build_prompt(system_instruction, similar_chunks, query)
response = call_llm(prompt)
```

A well-tuned RAG pipeline with 5-10 retrieved chunks often outperforms a "stuff everything in" approach even when the model could technically fit all the data. Less noise means stronger signal.

### Strategy 3: Sliding window with overlap

For tasks that process sequences (analyzing a long conversation log, reviewing a codebase file by file) use a sliding window that maintains overlap between segments. Process tokens 0-100K, then slide to 80K-180K (20K overlap), and merge results.

The overlap means content near window boundaries isn't missed. This works for tasks where each analysis unit is mostly local. Finding bugs in individual files, extracting entities from sections of text, or summarizing a transcript chapter by chapter.

### Strategy 4: Context compression

Before sending text to the model, strip what doesn't matter. Remove boilerplate, headers, footers, repeated sections. Use a cheap model to pre-summarize verbose sections. Strip HTML tags and markdown formatting when the model doesn't need markup.

A 100K-token document might compress to 20K tokens with minimal information loss.

### Strategy 5: Hierarchical context management

For production applications, use a three-tier system:

- **Tier 1 (always present):** System prompt, critical instructions, user preferences. 2-5K tokens.
- **Tier 2 (session context):** Current conversation summary, active task state. 5-20K tokens.
- **Tier 3 (on-demand):** Retrieved documents, code files, database results. Loaded when relevant.

This keeps the expensive, volatile content from hogging context space and ensures the most important instructions never get pushed out.

## RAG vs long context: picking the right approach

The decision comes down to one question: does the model need to reason across all of this data simultaneously?

**Use full context when:**
- You're comparing three vendor proposals and need to find contradictions across documents
- You're analyzing a contract where clauses in section 2 modify obligations in section 12
- You're reviewing a complete codebase for architectural patterns that span files
- The total data fits within the window with room for the response

**Use RAG when:**
- Your document collection is larger than any context window
- Queries are specific and targeted (not broad synthesis)
- Cost matters and you're making many queries per document
- Accuracy needs to stay stable as the data grows (long-context models degrade past ~32K tokens)

In practice, most production systems use both. Long context as working memory, RAG as knowledge retrieval. The line between the two keeps blurring, and that's a good thing.

## Common context window problems and fixes

**"My bot forgets the start of conversations."** Your context window is full and the model is silently dropping old tokens. Summarize the conversation so far, start a new session with the summary, or upgrade to a model with a larger window.

**"My API bill is way higher than expected."** Check your per-message input token count. If you're sending full conversation history with every turn, that's the culprit. Also check whether classification or extraction tasks are running through an expensive model.

**"My prompt got truncated silently."** The model hit its output token limit before finishing. Either your max_tokens setting is too low, or your prompt plus expected response exceeds the context window.

**"The model ignores information I gave it."** The relevant text is probably in the middle of a long input where attention is weakest. Move the important information to the beginning or the end of the prompt.

**"My local model runs out of VRAM."** Context windows plus large models consume GPU memory fast. Running a 70B parameter model at full 128K context can need 80+ GB of VRAM depending on quantization and architecture. Solutions: use a smaller model, quantize to 4-bit or 8-bit, or switch to RAG so you don't need the full context window.

## FAQ

**What happens when the context window is full?**
Most chat interfaces silently drop the oldest tokens. API calls typically return an error, but some platforms silently truncate. In either case, the model can't access text outside the window. It's gone as far as the model is concerned.

**Which LLM has the largest context window?**
Google's Gemini models offer up to 2 million tokens on Gemini 1.5 Pro (with reduced quality at the extremes). Among currently accessible models, Gemini 2.5 Pro, GPT-5.5, Claude Opus 4.7, DeepSeek V4-Flash, and Llama 4 Maverick all offer 1M-token windows.

**Does a larger context window mean better results?**
Not necessarily. Bigger windows introduce the lost-in-the-middle problem and cost more to fill. For tasks under 50K tokens, a 200K-window model on Claude Sonnet often outperforms a 1M-window model on Gemini Flash. Context capacity and model capability are separate dimensions.

**Can I increase a model's context window?**
No. The context window is fixed by model architecture. You can choose a model with a larger window or use workarounds like summarization, RAG, and chunking to simulate larger effective context.

**How do I check my token count?**
Use `tiktoken` for OpenAI models, Anthropic's token counting API for Claude, and Gemini's `count_tokens()` method. As a rough estimate: divide English word count by 0.75.

**Does context window size affect hallucination rates?**
A bigger window can help by providing the model with more reference material, but it can also hurt by introducing noise and diluting attention. Curated, relevant context matters more than total context size.

---

*Token pricing and context window sizes change frequently. Verify current numbers with your provider before making architecture decisions.*

---

## Related topics

- [What are AI tokens? A complete guide to LLM tokenization, costs, and context windows](https://aioutlooks.com/what-are-ai-tokens/). The hub article for this cluster.
- [What is a large language model?](https://aioutlooks.com/what-is-a-large-language-model/). Foundational understanding of the transformer architecture behind context windows.
- [How to stop wasting tokens in Claude Code: 7 data-backed fixes](https://aioutlooks.com/how-to-stop-wasting-tokens/). Practical optimization for the most common developer environment.
- [Building multimodal RAG with Gemini File Search](https://aioutlooks.com/build-multimodal-rag-with-google-file-search/). Implementation guide for the RAG strategies discussed above.
- [The illusion of AI reasoning](https://aioutlooks.com/the-illusion-of-ai-reasoning/). How attention mechanisms shape what models really do with context.
- Lost in the middle: why LLMs forget and how to work around it *(coming soon)*
- Prompt caching: the 90% cost reduction trick most developers miss *(coming soon)*
- Handling LLM token limits in production: a practical guide *(coming soon)*
