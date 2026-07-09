# Good Data Analysis

*The habits of careful analysts, re-versioned for the agentic era.*

In 2019, Patrick Riley published [Good Data Analysis](https://developers.google.com/machine-learning/guides/good-data-analysis), a distillation of what Google's most credible analysts actually did. I keep returning to it, because almost nothing in it has aged. What has changed is who does the typing.

In 2026 an agent writes the SQL, runs it, charts the result, and narrates a conclusion in less time than it takes to read this paragraph. Analysis has become nearly free to produce and exactly as expensive to get right. Wrong answers now arrive faster, in confident prose, with a tidy chart attached. That makes Riley's guide more relevant, not less: when the doing is delegated, what remains of the analyst's job is precisely what the guide was always about – validation, skepticism, and judgment.

This is my re-version. The core survives, compressed and re-grounded in the work I do (web analytics: GA4, BigQuery). A fourth section covers what the original could not have: how to delegate analysis to agents without losing the plot.

- **[Technical](#technical)** – how to examine data
- **[Process](#process)** – how to run an analysis
- **[Mindset](#mindset)** – how to be right more often
- **[Agents](#agents)** – how to delegate without losing the plot

## Technical

### Look at distributions, not summaries

Means and medians compress away exactly the things that matter: multimodal behavior, heavy tails, a second population hiding inside your data. Plot the histogram, the CDF, or a Q-Q plot before you quote an average. A "4-minute average session" reads very differently once you see it is two humps: humans at 40 seconds and a bot farm pinned at 30 minutes.

```
 sessions
    │
    │   █
    │  ███
    │  ███                                  ██
    │ █████                                ████
    │ █████                                ████
    │███████▄▄                          ▄▄██████▄▄
    └──────────────────────────────────────────────▶ duration
     0s  40s                               30m
       humans                            bot farm

     mean: 4 minutes. Nobody is at 4 minutes.
```

### Treat outliers as canaries

Outliers are often the first visible symptom of a problem with the analysis itself. Pages with impossibly high click-through usually mean double-counting; pages with zero clicks usually mean a broken tag. It is fine to exclude outliers or bucket them as "unusual", but only after you know why they ended up there. Some will never be explainable – time-box the hunt.

### Noise does not average itself away

"With this much data, the noise cancels out" is a comfortable lie. Every number you publish should carry some notion of confidence – an interval, a p-value, or at minimum the day-to-day variation that shows how much the metric wobbles on its own.

### Read raw examples

You cannot trust analysis code – yours or an agent's – until you have watched real rows flow through it. Summaries abstract; individual examples keep the abstraction honest. Sample deliberately: a few from every class if you are classifying, the extremes and the middle if you are measuring. (You do know what your distribution looks like, right?)

### Slice

Split by device, channel, country, browser, day. If the phenomenon could plausibly differ across subgroups, slicing is how you find out. If it should not differ, consistency across slices is cheap reassurance that you are measuring the right thing. When comparing two groups, watch for mix shifts – if the slices are weighted differently in each group, Simpson's paradox will happily invert your conclusion.

### Ask whether it matters, not just whether it is significant

With enough data everything is statistically significant; the question is whether a 0.1% difference changes any decision. The reverse trap exists too: with little data, "not significant" is not the same as "neutral". Ask how large an effect could still be hiding inside your confidence interval.

### Slice by time

Systems drift. Instrumentation that was verified at launch quietly breaks in month three, and nobody re-checks. Day-level slices catch the breakage and give you an honest feel for natural variation. When a day looks bizarre, do not delete it – use it as a hook to find the cause first.

### Count what you filter

Nearly every analysis begins with exclusions: internal traffic, bots, spam, out-of-scope users. State every filter and count what it removes. The cleanest way is to compute your metrics for the excluded population too. In my traffic reporting, the spam share is a headline number of its own, never a silent WHERE clause.

### Define your ratios

The interesting choices hide in denominators. "Sessions per user" – users with a session, users seen this month, or all registered accounts? Conditional metrics deserve the same care: "time to click" means "time to click, given a click", and the given part can shift between the groups you compare.

## Process

### Separate validation, description, and evaluation

Three stages, in order of increasing debate:

1. **Validation** – do I believe this data was collected correctly and means what I think it means?
2. **Description** – what does it objectively say? ("Sessions from paid social fell 12% week over week.")
3. **Evaluation** – so what? Is this good or bad, and for whom?

Description is where agreement lives; evaluation is where argument belongs. Blur them and you will reliably see the interpretation you were hoping for. You will jump between stages as you work – just always know which one you are in.

### Understand how the data got collected before reading it

If it came from an experiment, read the configuration and try the feature yourself. If it came from new instrumentation, learn how the events fire. Check the calendar for holidays and launches. My standing example: GA4's BigQuery export finalizes about two days late, so "yesterday" in the warehouse is a different question from "yesterday" in the UI.

### Check what should not have changed

Before answering the question you care about, rule out the changes you did not expect: user counts, error rates, segments the change should not touch. If the untouched things moved, your experiment (or your pipeline) has a problem more urgent than your hypothesis.

### Standard metrics first, custom second

New features come with tempting new metrics. Look at the boring ones first – sessions, clicks, revenue – even when you expect no change there. Standard metrics have survived years of scrutiny; your custom metric was born yesterday. When they disagree, the custom metric is usually the one that is wrong.

### Measure twice, differently

For anything new, compute the same quantity by two routes, ideally from two data sources. Agreement does not prove you are right, but disagreement reliably finds bugs: in logging, in filters, in your understanding.

### Check for reproducibility

An effect that is real shows up across populations, across time, and across random subsamples. A model that falls apart under small perturbations of the data has not captured anything fundamental about the process that generated it.

### Compare against what was measured before

If past analyses say mean page load is 2 seconds and yours says 5, you may be right – but you are guilty until proven innocent. You do not need exact agreement with history, just the same ballpark. Most surprising numbers turn out to be errors, not discoveries.

### Point new metrics at old truths first

A new satisfaction metric earns trust by confirming what you already know: that your best features score well and your known failures score badly. A metric that has never predicted a known outcome has no business announcing an unknown one.

### Hypothesize, then hunt for the evidence against

Analysis is iterative: you will see an anomaly, and a theory will suggest itself. The theory is not the finding. Ask what else must be true if the theory holds, then check. A learning effect should be strongest among heavy users. A launch effect should touch only the launched population, at the expected magnitude. Good analysis tells a story; the discipline is telling the story to yourself and then trying to break it.

### Iterate end-to-end

Get a rough answer through the entire pipeline before perfecting any stage. The view from the end changes your idea of what the beginning should do. Note the shortcuts you took – filters skipped, weird rows ignored – rather than fixing them all up front. A full first pass now costs minutes, so there is no longer any excuse for a week spent polishing stage one.

### Watch for metrics fed back into the system

If clicks feed the ranking system, more clicks cannot prove users are happier: the system was told to produce clicks. Never evaluate a change by a metric the change manipulates, and be wary even of slicing on fed-back variables – the mix shifts become impossible to reason about.

## Mindset

### Start with questions, not data or tools

There is always a reason someone wants an analysis. Force it into the shape of a question before touching data. Analysis without a question ends up aimless, and the failure mode has teeth now: an agent will generate unlimited aimless exploration on request. The mirror trap is just as old – a favorite technique in search of problems it happens to fit.

### Be champion and skeptic at once

When you find something interesting, run both prompts against yourself: what other evidence would show this is real, and what would invalidate it? The skeptic matters most when the person who asked wants a particular answer.

### Correlation still is not causation

Given enough metrics and enough experiments, some signals will align by chance. When you want to claim "A causes B", or even "A is a durable proxy for B", ask how you would validate it and whether a hidden C explains both. Be explicit with your audience about which claims you can and cannot make.

### Peers first, stakeholders second

A peer reviewer has no agenda; a stakeholder always does. Early on, peers know the gotchas and the prior work. At the end, they spot the oddities you have gone blind to. An adversarial agent review is a good filter to run before taking a peer's time, but it is a filter, not a substitute: the peer knows the context nobody wrote down.

### Say "I don't know" early, and own mistakes fast

Admitting the limits of your certainty feels like weakness and compounds into credibility. Discovering your own error and reporting it before anyone else finds it is one of the few reliable ways an analyst earns trust.

## Agents

The original guide assumed that doing the analysis was the expensive part. That assumption died. This section is what I have learned running analysis through agents daily.

### Plausible is the default output

A language model's native talent is producing what an answer should look like. For prose that is often enough; for analysis it is the failure mode. An agent's number arrives with the same confident formatting whether it is right or subtly filtered into fiction. Calibrate accordingly: fluency is not evidence.

### Correctness lives in the context

Most agent analysis errors are not SQL bugs; they are semantic bugs. The agent did not know that a flag arrives as a string, that the export lags two days, that the source table mixes real campaign tags with synthetic values that must be excluded. It cannot know what nothing told it. Document your schema's meanings, definitions, and traps once, put that in front of every agent, and every future analysis inherits the correction. Undocumented semantics are the single biggest source of confident wrong answers.

### Demand the working, not just the answer

Require three things with every result: the exact query that ran, row counts through each filter step, and a handful of raw examples. This is the old advice – count your filtering, look at examples – converted into a delegation contract. An answer that cannot show its working is a rumor.

### Get two answers by two routes

"Measure twice" used to be expensive discipline; now a second independent computation costs a sentence. Fresh agent, no shared context, ideally a different source or method. When the numbers agree you have learned a little; when they disagree you have almost certainly caught a bug you would otherwise have published.

### Never let the agent grade its own homework

An agent reviewing its own analysis inherits its own assumptions, and will bless them. Verification must be independent: at minimum a fresh context, better a different route, best a human who reads the working. Self-review catches typos, not worldviews.

### Spend the dividend on validation

When analysis gets ten times cheaper, the temptation is ten times more analyses. Mostly wrong. The scarce resource was never analyses; it was true ones. Hold volume roughly constant and spend the savings on the checks this guide describes – the ones nobody used to have time for.

### The question and the so-what stay yours

Agents collapse the middle of the job: given a question, they produce a description. The two ends do not delegate. Deciding what is worth asking requires knowing the business; deciding what the answer means requires owning the consequences. Riley's three stages map cleanly onto the division of labor: validation you enforce, description you delegate, evaluation you keep.

```
 the question ──▶ [ the analysis ] ──▶ the so-what
     yours           the agent's           yours
```

### You can't credibly orchestrate what you can't do yourself

Everything in this section assumes a reviewer who could, in principle, have done the work. That is the mechanism, not nostalgia. You can only spot the semantic bug if you know the schema, only judge the filters if you have written filters, only tell plausible from true if you learned the difference the slow way. This is why the first three sections survive the agentic era intact: the skills you no longer exercise daily are the qualification for supervising the agent that does. An analyst who cannot do the analysis cannot check it – only forward it.

## Closing

Riley closed by observing that careful work is invisible: nobody making decisions from your analysis sees that you checked population sizes or validated the effect across browsers. 2026 sharpens the point. Now even the analysis is invisible, produced in seconds by a machine. What remains visible – the only visible thing – is whether you were right. When anyone can generate an analysis by asking for one, being right is the entire job.

## Attribution

Adapted from [Good Data Analysis](https://developers.google.com/machine-learning/guides/good-data-analysis) by Patrick Riley (Google Machine Learning Guides, last major update June 2019), licensed under [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/). This version is substantially rewritten, compressed, re-grounded in web analytics, and extended with the Agents section and other material on agent-assisted analysis. Shared under the same license.
