---
title: 'microcode - the first fully RLM-powered terminal agent'
author: 'Farouk Adeleke'
date: '25 Jan 2026'
description: 'Microcode is a context-efficient terminal agent powered by Recursive Language Models (RLMs). How RLMs solve the context-rot problem and enable long-horizon coding tasks.'
tags: ['RLM', 'Terminal Agent', 'Context Management']
---

Microcode is a context-efficient terminal agent that excels in long horizon tasks and infinitely long prompts. It is fully powered by a Recursive Language Model.

CLI on Github: [Microcode Repository](https://github.com/modaic-ai/microcode)
RLM Engine via Modaic's `AutoProgram`: [Nanocode Repository](https://modaic.dev/farouk1/nanocode)

RLMs, or [Recursive Language Models](https://arxiv.org/pdf/2512.24601v1), introduced by Alex Zhang out of MIT CSAIL, aim to solve the **context-rot problem**. They propose a general inference strategy that

> "treats long prompts as part of an external environment and allows the LLM to programmatically examine, decompose, and recursively call itself over snippets of the prompt."

![RLM Graph](https://d1pz4mbco29rws.cloudfront.net/public/rlm-graph.png)

In short, instead of stuffing a potentially large zero-shot prompt into the context window, the prompt is treated as a variable in a REPL environment that the model can take sections of, write Python code to interact with, and so on. The model can also query smaller, faster sub-models (as code rather than a tool call, which is an important distinction) to increase the available context window. This allows the model to efficiently manage its own context instead of relying on the end user or developer. As I waited for [DSPy](https://dspy.ai/) to release its own module for using RLMs programmatically, I played around with the [v1 paper implementation](https://github.com/alexzhang13/rlm), and wondered how much coding agents could benefit from something like this.

## Why a terminal agent?

For most of 2025 and all of 2026, terminal agents have taken the world by storm, overtaking the IDE as the primary interface for coding agents. I've used Cursor, Windsurf, GitHub Copilot, and others, but as soon as I got my hands on Claude Code during the summer, I was immediately addicted. I'd attribute this to developers feeling more comfortable translating natural language to bash execution via a CLI than through a chatbot experience. Terminal agents' adoption was proof that filesystems are the best environment for agents to operate in. They are structured (treatable as a directed graph with versioning via git), human-interpretable, and can be interacted with in a way that LMs are most familiar with: code. Beyond coding tasks, coding agents have been successfully applied to general-purpose domains such as legal, medical, and finance. Numerous benchmarks, such as [Terminal Bench](https://www.tbench.ai/), have even emerged to evaluate their performance.

While a huge step forward, coding agents still have their vices. Long-horizon tasks that require understanding external documentation and holistic comprehension of a large codebase, while maintaining awareness of the original goal, are very hard due to **context-rot**. Context-rot is the principle that as more tokens are fit into the context window, performance degrades and attention breaks down. This is why you have to start new conversations with ChatGPT or Claude as your chats get longer. The reason Claude Code is so good, as revealed by someone who [reverse-engineered its system prompts](https://www.youtube.com/watch?v=i0P56Pm1Q3U&t=271s), is due to smart context engineering:

> "Beyond the initial system prompt, Claude Code actively inserts reminder blocks into the conversation history. Specifically, after task progression steps or calls to the to-do tool, a reminder about the to-do list's purpose and usage is added to the message history. This provides continuous reinforcement of key objectives throughout the interaction, contributing to robust task management."

Some other hacks include [context compression](https://factory.ai/news/evaluating-compression) and using subagents. RLMs are an intelligent way to turn tricks like this into a generalizable inference strategy, one where the model manages it's own context.

## The Microcode Stack

I started by taking a minimal implementation of a [coding agent introduced by an AI Lead at Ramp](https://github.com/1rgs/nanocode) and refactoring from there. Coding agents are basically just a model with access to tools such as...

- `read_file`
- `write_file`
- `edit_file`
- `glob_files`
- `grep_files`
- `run_bash`

...running until the task is deemed "complete" and then looping to the next user input. Building the RLM coding agent itself was fairly trivial, only like ~350 lines of code, since the DSPy team had released a composable [`dspy.RLM()`](https://x.com/isaacbmiller1/status/2013371005960401327) module by then. The DSPy version comes with a REPL and Python interpreter via [Deno & Pyodide](https://til.simonwillison.net/deno/pyodide-sandbox), tools to interact with sub-models, and instructions on how to use those tools and interact with variables in the REPL. They also expose a parameter in the RLM's contructor to add custom tools, which I used to give it coding agent abilities.

I then packaged the program on [Modaic](https://modaic.dev/farouk1/nanocode) for a clean separation of concerns between the AI system and the CLI built around it. Modaic allows you to run a packaged DSPy program and all it's required dependencies in one line of code via the `AutoProgram` primitive. This makes it easy to isolate the CI/CD and experimentation of the DSPy program separately from the traditional software. This makes per-user or per-codebase prompt optimization/fine-tuning/RL plug-and-play!!

```python
from modaic import AutoProgram

agent = AutoProgram.from_precompiled(
	"farouk1/nanocode", # modaic repository path to the RLM engine
	rev=os.getenv("MODAIC_ENV", "prod"), # conditional env for CI/CD
	config=config # configuration overrides to the base system
)
```

After that, I added some basic CLI functionality around it: switching models with OpenRouter, syntax highlighting, history management, and handling extremely large text pastes. Because of the external tools I introduced to allow the RLM to navigate a codebase or filesystem, it technically has two "environments" or action spaces. The first is its internal environment, where it reasons through the task in code; the second is my filesystem, where it can write bash commands, write to files, grep search, and so on. I was hesitant to add a `read_file` tool because that somewhat defeats the purpose of RLMs, but decided it would be simpler than finding a way to copy file contents into the Deno environment. The end result is still purposely minimal, ~1800 lines of code across one entrypoint file and some utilities, to promote flexibility for developers interested in adding their own additions. There's also a non-interactive mode for those who'd like to try their hand at running benchmarks.

I also added reasoning callbacks via the `--verbose` flag. This allows you to differentiate between actions taken within the internal REPL (dim grey) and the external filesystem (bright magenta). I find this helpful for collecting and scoring trajectories for further optimization on open or closed-sourced base models.

As a fun experiment, I gave two instances of Microcode a shared markdown file to have a conversation in an infinite loop. Because of the instructions to express actions as code, you can see the RLM determining whether it should respond based on whether the count of the other agent's messages is longer. What's especially cool is that I didn't have to explicitly prompt them to agree on this convention; I just told them their agent # and where they were having the conversation. They converged on this method completely on their own.

![RLM Conversation](https://d1pz4mbco29rws.cloudfront.net/public/rlm-conversation.png)

_[Full conversation here if you're interested](https://www.notion.so/RLMeo-RLMiet-2f4f6689b4c08045a38eede1d4fdb5e6?source=copy_link)_

The results of using Microcode across a variety of task difficulties were extremely promising. I used it to add documentation to some larger codebases I had lying around. At one point, I was using it to recursively add features to itself! For ultra-long tasks, it keeps the task as a variable in its REPL and uses a combination of file search and bash for actions in my filesystem. Outside of coding, Microcode is a general-purpose agent that excels at summarization tasks as well as pinpointing small details from a large corpus of text.

![RLM Chat](https://d1pz4mbco29rws.cloudfront.net/public/rlm-chat.png)

_Here, I tasked Microcode with finding an answer in a [23-page research paper](https://arxiv.org/pdf/2512.22245v1). It cost me roughly a cent to run._

| Timestamp        | Model                 | App     | Tokens | Cost ($) | Speed (tps) |
| ---------------- | --------------------- | ------- | ------ | -------- | ----------- |
| Jan 25, 06:23 PM | MiniMax M2.1          | liteLLM | 106    | 0.012    | 66.2        |
| Jan 25, 06:22 PM | Qwen3 Coder 480B A35B | liteLLM | 6,885  | 0.00226  | 107.5       |
| Jan 25, 06:22 PM | Qwen3 Coder 480B A35B | liteLLM | 4,394  | 0.00158  | 129.1       |
| Jan 25, 06:22 PM | Qwen3 Coder 480B A35B | liteLLM | 3,119  | 0.00103  | 127.4       |
| Jan 25, 06:22 PM | Qwen3 Coder 480B A35B | liteLLM | 2,166  | 0.000815 | 106.7       |
| Jan 25, 06:22 PM | Qwen3 Coder 480B A35B | liteLLM | 1,507  | 0.000634 | 121.2       |
| Jan 25, 06:22 PM | Qwen3 Coder 480B A35B | liteLLM | 1,159  | 0.000383 | 17.5        |

_OpenRouter logs for this run_

I think this will be a great v1 for production applications of RLMs as coding agents, and I'm excited to see what the community does with it. Next step is throwing it into Terminal Bench.
