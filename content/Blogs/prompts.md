---
title: 'prompts are eating the world'
author: 'Farouk Adeleke'
date: '14 Oct 2025'
description: 'Software is changing again, not incrementally, but fundamentally. This post sums up why declarative AI programming (and our intuition about DSPy) is an eventual next step in the software evolution, and why open infrastructure like Modaic must exist to enable it.'
tags: ['Theory']
---

## TLDR

After decades in practice, we've seen two very interesting shifts happen close together: from explicit code to neural networks, and now to programmable language models (as agents or workflows). This post aims to explain why declarative AI programming (and our intuition about DSPy) is an eventual next step in the software evolution, and why open infrastructure like Modaic must exist to enable it.

---

# I. Software's Essence

To preface, I'll assume you already know what software is, but for those who don't, it's instructions that tell computers how to manipulate or mutate information and automate tasks. For a long time, these instructions took the form of code: humans translating their intent into increasingly higher-level programming languages, from machine code to assembly to C to Python. More formally, software is a set of computer programs that instruct the execution of a computer. For the sake of this blog, we can apply this definition to the term **Declarative AI Software**, which is a set of **Declarative AI Programs**, compiled by a declarative framework like [DSPy](https://dspy.ai/) or Microsoft's [Trace library](https://microsoft.github.io/Trace/).

# II. AI Software Will Eat Labor

Marc Andreessen wrote that "**software is eating the world.**" But as Ben Horowitz points out in his a16z talk, software (relatively) hasn't actually replaced that much at all.

<iframe width="850" height="500" src="https://www.youtube.com/embed/dhyhR4Bzc0I?si=ZTojuyjwp_-oXjot&start=204" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen class="w-full rounded-md my-12"></iframe>

To frame this a bit better, consider this:

- Worldwide SaaS market: ~$300 billion/year
- US labor market alone: **$13 trillion/year**

The gap is huge. As Ben Horowitz quoted, **almost every software company to date has essentially taken a "filing cabinet" and turned it into a database.** Sabre's airline reservation system in the 1960s and Epic's electronic health records to Workday's HR systems are good examples of this. We've digitized the workflows (moved from paper to mainframe to cloud), but we haven't replaced the labor.

The filing cabinets were read by humans. The databases are read by humans. The medium changed from paper to mainframe to cloud, but the humans remained. The phrase for this is human-in-the-loop automation.

Take Zendesk as an example: 1,000 support agents (humans) answering questions costs roughly 75 million per year. Zendesk itself? Just 1.4 million. The software cost is a rounding error compared to the labor cost. Each answer costs ~38 in human labor, 0.69 in software.

If AI can answer those questions end-to-end, Zendesk's revenue for this customer could go to zero (no seats needed), or it could 3x to 5 million by charging outcome-based pricing while the customer saves 70 million. This is, of course, under the assumption Zendesk is able to successfully implement agents capable enough for this task.

"Software eats labor" means that instead of just digitizing the workflows, we're now digitizing the labor with AI components that can act on the behalf of a human. Its also interesting to note that instead of training one massive model to do everything, we're composing multiple LLM calls, retrievers, tools, and logic into coherent multi-agent systems. A layer of traditional software over the AI components. It's very software-esque.

# III. AI applications as an Operating System

The problem facing a large majority in this space is how to frame the AI landscape, both within their own systems and in a infrastructure sense. A good mental model I've found very helpful, and a consensus I'm starting to see more of, is to think of AI applications like a metaphorical operating system. We interact and delegate tasks to an OS similarly to how we interact with an LLM. You also see a lot of AI infra startups popping up to support components of this hypothetical OS.

Lots of strong analogies can be drawn:

- **LLM = CPU**: The processing unit that executes instructions
- **File System or Embeddings = Disk**: Where data is stored and is accessed at runtime
- **Context Window = RAM**: Temporary structured data access during inference
- **Prompts = Code**: Instructions that program the system

![LLM OS - Andrej Karpathy](https://pbs.twimg.com/media/F-nOa_rboAAqe0o?format=png&name=medium)
_Andrej Karpathy's visualization of an "LLM OS" ([link](https://x.com/karpathy/status/1723140519554105733))._

And like any computer architecture, it needs its own programming paradigms, development practices, and infrastructure, none of which has been fully standardized yet.

# IV. Software 3.0: Prompts Are Eating Through the Stack

<iframe width="850" height="500" src="https://www.youtube.com/embed/LCEmiRjPEtQ?si=98ttQc4qEnYBOXzF&start=250" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen class="w-full rounded-md my-12"></iframe>

Andrej Karpathy observed the first shift to a new type of software "eating through the stack" firsthand at Tesla. The Autopilot team started with tons of C++ code (Software 1.0) and some neural networks for image recognition (Software 2.0). Over time, the neural networks grew in capability and systematically replaced the C++ code. Software 2.0 literally ate through the Software 1.0 stack on GitHub.

Now we're seeing it again.

- **Software 1.0**: Deterministic lines of code you write directly (C++, Python, etc.)
- **Software 2.0**: Neural network weights - you don't write them, you tune them by optimizing on datasets.
- **Software 3.0**: LLMs programmable with prompts written in English. If you look through the codebase of your average AI startup, you'll find thousands of lines worth of prompt strings, as well as scaffolding around API calls to an LLM provider.

**Note:** Neural networks went from being fixed-function computers (image -> categories) to being **programmable computers**. Your prompts are programs, but for the most part, manual prompt-tuning is a static process.

# V. The Fuzzy State of Prompts

The problem is that prompts (and AI Engineering as a whole) are in a fuzzy, pre-scientific state.

Prompt engineering today looks something like...

- 10,000+ tokens in just system prompts.
- **Threatening** the model to do things the right way.
- Weird tricks to try to hack the model's responses.
- Endless trial and error with no systematic methodology (what I like to call **vibe prompting**). Yes, even with evals.

**Prompt engineering is not a science.** It's not repeatable, reproducible, or composable across models or workflows. Why? Because the English language is probably the worst medium we could have thought of for conveying instructions to a machine. It's full of ambiguity, context-dependency, and nuance that is hard to capture in a machine-readable format. Tech Twitter would describe this as "noise".

When you talk to a human, you adjust your communication style, add emphasis, micromanage maybe. For some reason, our current standards for prompt engineering apply human communication patterns to machines.

# VI. A Necessary Abstraction

A glimmer of hope is forming though. Prompt optimizers like GEPA, OPRO and DSPy's MIPRO(v2) represent a step in the right direction: they offer systematic, algorithmic approaches to generating and refining prompts instead of manual iteration.

This is the natural evolution. Just as we moved from manually flipping bits to using compilers, or manually deriving the gradient formulas to using .backwards() in PyTorch, we're moving from manually crafting prompts to using optimizers. **Necessary abstractions** open up implementation (and therefore innovation) to people who aren't subject matter experts. It makes it so that you aren't...

**"Maintaining code at a lower abstraction than you have to"**

Omar Khattab - Creator of DSPy

# VII. DSPy: The Compiler for the LLM OS

What is DSPy? It stands for "**D**eclarative **S**elf-improving **Py**thon". Think of it as the compiler that bridges:

**Human Intent -> DSPy Signatures (Input/Output Definitions) -> Optimized Prompts -> LLM Execution -> Task Completion**

DSPy forces you into strategic abstractions so that you are required to provide well-defined specifications for your systems. This is code.

![Hacker News Clipart](https://modaic.dev/comic-clipart.png)
_From a Hacker News discussion about declarative programming ([link](https://news.ycombinator.com/item?id=41547841))._

You notice a similar abstraction in the introduction of assembly language. It bridged human intent to machine code without requiring programmers to manually write 1s and 0s.

In DSPy, you don't write prompts. You write **signatures** that specify input or output behavior, select **modules** that define certain prompting strategies, and run **optimizers** that automatically compile your specifications into effective prompts and weights. The result is programs that are way more reliable, maintainable, and portable across models. Programs that improve over time as better optimizers become available and new models are released.

# VIII. From Programs to Software

To reiterate, a collection of programs (or declarative AI programs) assembled to complete a business function is called **software**.

And where there exists software, there exists a development lifecycle, a set of rules and standards that ensures your software systems are improving over time.

- **Software 1.0** -> SDLC (Software Development Lifecycle)
- **Software 2.0** -> MLOps (Machine Learning Operations)
- **Software 3.0** -> Declarative AI Ops or AI Software Lifecycle

Each paradigm also brings its own class of engineer to maintain this lifecycle:

- **Software 1.0** -> Software Engineer
- **Software 2.0** -> ML Engineer
- **Software 3.0** -> AI Engineer

# IX. Open Source as the Accelerant

Every major software paradigm has been accelerated by open source. The question is never "if" but "how" and "where".

Linux gave us one of the first open source operating systems, necessary abstractions over machine code. To maintain his project, Linus created Git (which later brought GitHub) which became the platform for open source software development. PyTorch and TensorFlow abstracted backpropagation and gradient computation when working with neural networks, lowering the barrier to entry and leading to an explosion in model innovation. Hugging Face was the ecosystem that allowed for the new components these abstractions created to be shared and composable. DSPy is introducing necessary abstractions over the prompting strategies and the LLMs themselves. It's a step towards a true democratization for building AI **applications or software**. The primitives are forming, but there is no ecosystem yet to collaborate on them.

# X. The Right Level of Abstraction

You can think of **primitives** as the smallest units of work for software. A good mental model is if diving a bit deeper into the component is a science of its own (or highly convoluted), then it's a primitive. For example, GEPA, one of the newer optimizers, is a primitive of Software 3.0 because it's necessary for the compilation of a successful declarative AI program. As an SDK, it's portable or reusable but if you wanted to maintain it yourself, you'd have to learn about Genetic-Pareto. Exa's web search tool is a primitive because their API is portable or reusable but the search algorithms under the hood are a science of its own.

The idea is that I shouldn't have to learn about Genetic-Pareto to use an optimizer, or learn about the search algorithms to use web search. I should just expect my systems to get better as the subject matter experts create better algorithms or model providers create better models.

The challenge is you need to abstract the right components for people to work with or you end up in a messy spot. **Too highly abstracted** or the **wrong abstractions** ([see OpenAI's agent builder](https://platform.openai.com/docs/guides/agents/agent-builder)) and you're left with slop. Bespoke solutions that can't handle real-world complexity, scale, or granularity. **Not enough abstraction** and only the experts can actually contribute. The barrier to entry stays really high.

A good balance is a platform that **lowers the barrier to entry while still relinquishing granularity and control** to users who want to reproduce, optimize, and innovate on the primitives. Code is usually the best medium for these type of things so it is unlikely canvas-style workflows will fill the spot.

For Software 3.0, these primitives include:

- **Environments**: The context in which agents operate, such as the tools they have access to.
- **Datasets**: Evaluation data from graded input or output pairs or sets of few-shot examples for the optimizers to learn from.
- **Metrics**: A systematic measurement of performance, what I like to call a "function of success" for the modules you implement.
- **Signatures**: Input or output specifications.
- **Optimizers**: Algorithms that compile specifications into prompts.
- **Models**: The LLM (or eventually SLM) being called and its configuration, such as temperature or weights.

# XI. Why a Developer Collaboration Platform for Software 3.0 Needs to Exist

The goal of developer collaboration platforms is to streamline the lifecycle of the type of software it captures. GitHub aids in streamlining the SDLC for traditional code and Hugging Face for MLOps. Modaic aims to aid in streamlining the Ops for declarative AI software. Open source within this ecosystem is usually a fortunate byproduct of these platforms aiding the development flow. Value is given to those who participate in that lifecycle and leverage already-built primitives.

**Note:** Open source accelerates innovation only when there's a medium for collaboration. You can use GitHub to store DSPy programs, but just like you can't reasonably host and download model weights on GitHub, someone wouldn't be able to use your programs unless they copied your code (which now leaves maintenance up to them).

---

## Further Reading

- [Ben Horowitz: Software is Eating Labor](https://www.youtube.com/watch?v=dhyhR4Bzc0I)
- [Andrej Karpathy: Software 3.0](https://www.youtube.com/watch?v=LCEmiRjPEtQ)
- [Harvard: The Value of Open Source Software](https://www.hbs.edu/ris/Publication%20Files/24-038_51f8444f-502c-4139-8bf2-56eb4b65c58a.pdf#page=31.22)
- [DSPy: Compiling Declarative Language Model Calls into Self-Improving Pipelines](https://arxiv.org/abs/2310.03714)
- [On Engineering AI Systems that Endure The Bitter Lesson - Omar Khattab, DSPy and Databricks](https://www.youtube.com/watch?v=qdmxApz3EJI)
- [Introducing Modaic](https://modaic.dev/us)
