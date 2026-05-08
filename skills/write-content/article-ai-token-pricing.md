# AI token pricing guide: what LLMs actually cost per model (May 2026)

I know someone who burned $3,400 in API costs in a single weekend. An autonomous coding agent got stuck in a refactoring loop: modify a file, compile, fail, try again, repeat. Each iteration cost a few cents. Nobody noticed until the Monday morning billing alert.

The agent wasn't using an expensive model. It was just running too many calls. A hundred thousand iterations at $0.03 each. That's $3,000. The rest was the server it was running on.

This is AI token pricing in the real world. Not the tidy per-million-token rates on the pricing page. The messy reality where a configuration you forgot about, a loop you didn't anticipate, or a model selection you didn't reconsider costs you a month's rent while you sleep.

This guide compares actual token costs across all major providers, shows you how to estimate what your application will cost before you build it, and gives you the routing strategies that cut bills without cutting quality.

## Quick verdict: the cheapest model for every use case

**Just need the cheapest option for simple tasks?** DeepSeek V4-Flash at $0.14 per million input tokens, $0.28 per million output. Of the frontier API providers covered here, it's the cheapest on output by a wide margin, but quality varies. Older models and self-hosted options can be cheaper. Test your specific task before committing.

**Want a balance of quality and cost?** Gemini 2.5 Flash at $0.30/M input, $2.50/M output. Built for speed, handles complex reasoning decently, has a 1M-token context window. Or Claude Haiku 4.5 at $1/M input, $5/M output. Costs more but the output quality is consistent. For the budget option, Gemini 2.5 Flash-Lite at $0.10/M input, $0.40/M output is the cheapest Google model.

**Building something that needs real intelligence?** Claude Sonnet 4.6 at $3/M input, $15/M output. The sweet spot for coding, analysis, and multi-step reasoning. Most production work lives here.

**Doing architecture, complex debugging, or agentic work?** Claude Opus 4.7 at $5/M input, $25/M output. Or GPT-5.5 at $5/M input, $30/M output. Use these sparingly. Reserve for the 20% of tasks that genuinely need them.

---

## At-a-glance pricing (May 2026)

All prices are per million tokens, standard API rates. Output costs almost always dominate your bill. Generating text costs roughly 4-6x more than reading it.

<table>
  <thead>
    <tr>
      <th>Model</th>
      <th>Provider</th>
      <th>Input per 1M</th>
      <th>Output per 1M</th>
      <th>Context window</th>
      <th>Cached input</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>GPT-5.5</td>
      <td>OpenAI</td>
      <td>$5.00</td>
      <td>$30.00</td>
      <td>400K+</td>
      <td>$0.50</td>
    </tr>
    <tr>
      <td>GPT-5.4</td>
      <td>OpenAI</td>
      <td>$2.50</td>
      <td>$15.00</td>
      <td>400K+</td>
      <td>$0.25</td>
    </tr>
    <tr>
      <td>GPT-5.4-mini</td>
      <td>OpenAI</td>
      <td>$0.75</td>
      <td>$4.50</td>
      <td>128K+</td>
      <td>$0.075</td>
    </tr>
    <tr>
      <td>Claude Opus 4.7</td>
      <td>Anthropic</td>
      <td>$5.00</td>
      <td>$25.00</td>
      <td>1M</td>
      <td>$0.50</td>
    </tr>
    <tr>
      <td>Claude Sonnet 4.6</td>
      <td>Anthropic</td>
      <td>$3.00</td>
      <td>$15.00</td>
      <td>1M</td>
      <td>$0.30</td>
    </tr>
    <tr>
      <td>Claude Haiku 4.5</td>
      <td>Anthropic</td>
      <td>$1.00</td>
      <td>$5.00</td>
      <td>200K</td>
      <td>$0.10</td>
    </tr>
    <tr>
      <td>Gemini 2.5 Pro</td>
      <td>Google</td>
      <td>$1.25</td>
      <td>$10.00</td>
      <td>1M</td>
      <td>$0.125</td>
    </tr>
    <tr>
      <td>Gemini 2.5 Flash</td>
      <td>Google</td>
      <td>$0.30</td>
      <td>$2.50</td>
      <td>1M</td>
      <td>$0.03</td>
    </tr>
    <tr>
      <td>Gemini 2.5 Flash-Lite</td>
      <td>Google</td>
      <td>$0.10</td>
      <td>$0.40</td>
      <td>1M</td>
      <td>$0.01</td>
    </tr>
    <tr>
      <td>DeepSeek V4-Flash</td>
      <td>DeepSeek</td>
      <td>$0.14</td>
      <td>$0.28</td>
      <td>1M</td>
      <td>$0.0028</td>
    </tr>
  </tbody>
</table>

Two things jump out. First, the spread is massive: GPT-5.5 output costs $30/M while DeepSeek charges $0.28/M. The same work costs over 100x more per token. Second, output tokens are the real cost. At $15/M output on Claude Sonnet, generating a 500-token response costs $0.0075. Sounds like nothing until you multiply it by 10,000 conversations a day. That's $75/day just in output, $2,250/month. The input tokens are the cheaper half of the equation.

## How token pricing actually works

LLM APIs charge per million tokens, usually at different rates for what you send (input) and what the model writes back (output). Output costs more because generation is computationally heavier than reading.

A mid-priced model like Claude Sonnet charges $3/M input and $15/M output. That means:

- Sending a 2,000-token prompt costs $0.006
- Getting a 500-token response costs $0.0075
- Total per conversation turn: roughly $0.0135

Sounds manageable. Now multiply by 10,000 conversations a day. That's $135/day, $4,050/month for a single-turn bot. Add conversation history that grows every turn, and the input costs compound fast.

Some providers add long-context surcharges. OpenAI charges 2x the base rate when your input exceeds roughly 270K tokens. Google charges roughly 2x across all Gemini models for prompts above 200K tokens. These surcharges matter if you're working with full codebases or large document sets.

## The hidden costs most developers miss

**Reasoning tokens.** GPT-5.5, Claude with extended thinking, and Gemini reasoning all generate internal "thinking tokens" that you pay for but never see. A single complex query can burn 10,000 reasoning tokens before producing a 200-token visible answer. You pay for all 10,200 tokens at the output rate. On GPT-5.5 at $30/M output, that one query costs $0.31. For a query where a cheaper model would've given the same answer for $0.01. Whether that extra thinking actually improves output is a real question — explored further in the article on [the illusion of AI reasoning](https://aioutlooks.com/the-illusion-of-ai-reasoning/).

**System prompts are a recurring tax.** Every system prompt you send gets processed on every single call. A 2,000-token system prompt on Claude Sonnet at $3/M input costs $0.006 per call. At 10,000 calls per day, that's $60/day, $1,800/month, just for the system prompt. Shorten it. Twenty-five words of instructions work better than 250.

**Full conversation history is the silent cost multiplier.** The default behavior in most API integrations: include every previous message with every new turn. By turn 20, you're sending 19 prior messages the model already processed. Input costs triple or quadruple over a session. Summarize conversations periodically, replace old history with a 200-token summary, or start fresh sessions.

**The autonomous agent problem.** This is the $3,400 example from the opening. [AI agents](https://aioutlooks.com/what-is-an-ai-agent/) that can run in loops (coding, research, data processing) can silently burn through thousands of API calls per hour. A failure condition (infinite retry on a broken test, a parsing loop) creates a cost firehose. Always set a max iterations limit. Always set spend alerts on your account. And never let an agent run unsupervised overnight unless you've run it supervised through the same workload first.

## How to estimate what your app will cost

Before you write a line of code, calculate your expected cost. Here's the formula:

```
Cost per call = (input_tokens / 1,000,000 × input_rate) + (output_tokens / 1,000,000 × output_rate)
```

Or in Python, with a cost estimator you can use for any model:

```python
def estimate_cost(model, input_tokens, output_tokens, calls_per_day):
    # Pricing per 1M tokens (May 2026)
    rates = {
        "gpt-5.5":        (5.00, 30.00),
        "gpt-5.4":        (2.50, 15.00),
        "gpt-5.4-mini":   (0.75, 4.50),
        "claude-opus":    (5.00, 25.00),
        "claude-sonnet":  (3.00, 15.00),
        "claude-haiku":   (1.00, 5.00),
        "gemini-flash":   (0.30, 2.50),
        "deepseek-flash": (0.14, 0.28),
    }
    if model not in rates:
        raise ValueError(f"Unknown model: {model}")

    input_rate, output_rate = rates[model]
    cost_per_call = (input_tokens / 1_000_000 * input_rate) + (output_tokens / 1_000_000 * output_rate)
    daily_cost = cost_per_call * calls_per_day
    monthly_cost = daily_cost * 30

    return {
        "cost_per_call": round(cost_per_call, 5),
        "daily_cost": round(daily_cost, 2),
        "monthly_cost": round(monthly_cost, 2),
    }

# Example: a chatbot doing 10,000 convos/day, 2K input + 500 output each
result = estimate_cost("claude-sonnet", 2000, 500, 10000)
print(f"Per call: ${result['cost_per_call']}")   # ~$0.0135
print(f"Per day: ${result['daily_cost']}")        # ~$135
print(f"Per month: ${result['monthly_cost']}")     # ~$4,050
```

Now plug in your numbers.

Here's what different applications look like in practice:

<table>
  <thead>
    <tr>
      <th>Use case</th>
      <th>Model</th>
      <th>Tokens per call</th>
      <th>Calls/day</th>
      <th>Monthly cost</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>Support chatbot</td>
      <td>Claude Haiku</td>
      <td>2,000 in + 500 out</td>
      <td>5,000</td>
      <td>~$675</td>
    </tr>
    <tr>
      <td>Coding assistant</td>
      <td>Claude Sonnet</td>
      <td>8,000 in + 1,500 out</td>
      <td>2,000</td>
      <td>~$2,790</td>
    </tr>
    <tr>
      <td>Document classifier</td>
      <td>GPT-5.4-mini</td>
      <td>1,000 in + 100 out</td>
      <td>50,000</td>
      <td>~$1,800</td>
    </tr>
    <tr>
      <td>Research agent</td>
      <td>GPT-5.5</td>
      <td>15,000 in + 3,000 out</td>
      <td>500</td>
      <td>~$2,475</td>
    </tr>
    <tr>
      <td>Simple translation</td>
      <td>DeepSeek V4-Flash</td>
      <td>500 in + 300 out</td>
      <td>20,000</td>
      <td>~$92</td>
    </tr>
  </tbody>
</table>

The translation pipeline costs $92/month on DeepSeek. The research agent costs $2,475 on GPT-5.5. Same architecture pattern, different models, 27x cost difference. That's why [understanding what tokens actually are](https://aioutlooks.com/what-are-ai-tokens/) and how they're priced isn't optional. It's the difference between a profitable product and a money pit.

## Model routing: match the job to the cheapest model that does it

Here's something that took me too long to learn: the most expensive model doesn't give the best results. It gives the same result, only slower and more expensively, for most tasks.

Route your requests:

**Classification, extraction, summarization, translation, and simple Q&A** → the cheapest model that works. Test GPT-5.4-mini, Gemini Flash-Lite, or DeepSeek. These tasks don't need frontier reasoning. A model that's 95% as accurate at 1/30th the cost is the correct choice.

**Code generation, content writing, and multi-turn conversations** → mid-tier models. Claude Sonnet or GPT-5.4. The jump from Haiku to Sonnet matters here; output coherence and instruction following improve measurably.

**Architecture decisions, complex debugging, and multi-step reasoning** → expensive models. Claude Opus or GPT-5.5. Reserve these for the 20% of prompts where the extra reasoning actually changes the output.

Not every prompt needs GPT-5.5. A 80/20 routing split (80% cheap, 20% expensive) produces near-identical results at roughly a quarter of the cost of routing everything to the expensive model.

## Prompt caching: the cost reduction most developers skip

Prompt caching cuts input costs by 90% on repeated content. Mark content that repeats across requests (system prompts, tool definitions, background documents) and pay cache-read rates instead of full input rates.

On Claude Opus, that means $5/M input drops to $0.50/M for cached reads. On GPT-5.5, $5 drops to $0.50. DeepSeek charges $0.0028 for cached reads. Nearly free.

Caching works best for:
- Multi-turn conversations (same system prompt every turn)
- Agent tool calls (same tool definitions every cycle)
- Document Q&A (same document, different questions)
- Batch processing (same instructions, different inputs)

It doesn't help for unique one-off prompts. No point marking a cache breakpoint if the content changes every request.

[Optimizing token usage](https://aioutlooks.com/how-to-stop-wasting-tokens/) goes further, but prompt caching alone can halve your input costs.

## Five strategies to cut your bill this week

Not theoretical. Things you implement in an afternoon.

**1. Route by model complexity.** Swap classification, extraction, and summarization from Claude Sonnet to Haiku or Gemini Flash. Save 50-70% on those calls with no perceptible quality drop for simple tasks.

**2. Enable prompt caching.** Mark your system prompt and shared context as cacheable. It takes one initial request to warm the cache at the standard rate. After that, every read drops 90% in cost. The cache stays alive for 5 minutes and resets on each hit.

**3. Stop sending full conversation history.** After 10 turns, summarize the conversation into a 200-token summary. Replace the history with the summary. Input tokens per turn drop from 20K+ to roughly 3K.

**4. Cap your output tokens.** If you're extracting a single field from a document, set `max_tokens` to 50. Don't let the model write 500 words about why it chose the answer. You pay for every one of those words.

**5. Batch non-urgent work.** OpenAI's Batch API gives 50% off for asynchronous processing. Anthropic and Google have similar programs. If a task can wait a few hours, batch it and cut the cost in half.

None of these are hard. Together they can cut a typical $5,000/month API bill below $1,000.

## FAQ

**Which AI model is the cheapest overall?**

Of the frontier API models covered in this guide, Gemini 2.5 Flash-Lite at $0.10/M input, $0.40/M output. DeepSeek V4-Flash at $0.14/M input, $0.28/M output. Older generation models like Gemini 2.0 Flash-Lite ($0.075/M input) and self-hosted open-source models can be cheaper still but trade capability and convenience for that lower price. Test your specific workload. The cheapest model isn't always the best value if you need two calls to get a correct answer.

**How much does 1,000 tokens actually cost?**

On GPT-5.5: $0.005 for input (1000/1M × $5). On DeepSeek: $0.00014 for input. Over a million API calls, that's the difference between $5,000 and $140. A penny per call adds up.

**Why are output tokens more expensive than input?**

Generating text requires forward passes through every layer of the model. Reading (encoding) can be parallelized more efficiently. Multiply by the GPU time difference and you get the 4-6x output premium.

**Does prompt caching work on every provider?**

Anthropic and OpenAI both support it. DeepSeek has it at extremely low rates. Google's Gemini platform supports context caching with its own pricing structure. Check each provider's docs for minimum token thresholds. Anthropic needs at least 1,024-4,096 tokens depending on model to trigger caching.

**How much do reasoning models really cost?**

GPT-5.5 at $30/M output: a query that generates 10,000 thinking tokens plus a 200-token answer costs $0.31. Claude Opus 4.7 at $25/M: similar query costs $0.26. A standard model on the same prompt might cost $0.01. Use reasoning models only when the extra thinking actually changes the answer.

---

*All pricing as of May 2026. Check your provider's pricing page before budgeting. Rates change and long-context surcharges vary.*

---

## Related topics

- [What are AI tokens? A complete guide to tokenization, costs, and context windows](https://aioutlooks.com/what-are-ai-tokens/). The hub for this cluster.
- [LLM context windows explained: limits, costs, and developer workarounds](https://aioutlooks.com/llm-context-windows-explained/). How context size affects your costs.
- [What is an AI agent?](https://aioutlooks.com/what-is-an-ai-agent/). How autonomous agents work and why they can burn through API costs.
- [The illusion of AI reasoning](https://aioutlooks.com/the-illusion-of-ai-reasoning/). Whether reasoning models actually think or just pattern-match.
- [How to stop wasting tokens in Claude Code: 7 data-backed fixes](https://aioutlooks.com/how-to-stop-wasting-tokens/). Optimization for the most common dev environment.
- Prompt caching: the 90% cost reduction trick most developers miss *(coming soon)*
- Thinking tokens explained: what reasoning models actually cost you *(coming soon)*
