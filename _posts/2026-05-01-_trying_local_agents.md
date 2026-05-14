---
layout: post
title: "Trying Local Agents"
date: 2026-05-13
categories: [agents]
tags: [openclaw, ollama, agents]
author: Lucas
---

# Main Goal

The main goal of this project was to get first contact with local agents. I wanted to understand how OpenClaw works before installing it on my main home server.

The use case was not to build a chatbot for fun. I wanted to automate small daily tasks that need some reasoning and are not easy to solve with a normal deterministic script. My first idea was a webwatcher agent that checks apartment listings near me and stores useful findings.

At the same time I wanted to keep privacy and security in mind. If an agent can use tools, read files, and write files, I do not want to just run it directly on my server without understanding the boundaries first.

# First Setup

I started by installing OpenClaw and Ollama on the server. The setup itself was simple and used the install scripts from both projects:

```bash
curl -fsSL https://openclaw.ai/install.sh | bash
curl -fsSL https://ollama.com/install.sh | sh
```

After that I downloaded a small local model and linked it in the OpenClaw config. Most of my testing used `ollama/gemma4:e2b`.

I also used `jq` to format the OpenClaw config file while editing it:

```bash
jq . ~/.openclaw/openclaw.json > /tmp/fixed.json
mv /tmp/fixed.json ~/.openclaw/openclaw.json
```

New agents can be added with:

```bash
openclaw agents add webwatch-local
```

This is not meant as a full tutorial. These are just the main commands I used to get started and to change the setup while testing.

# Local Model

One important goal was to run simple tasks locally. My server has a GTX 1070 and 16 GB of RAM, so I see it more like an edge device than a powerful AI machine.

The small Gemma model was able to fit on the GPU at first. But when OpenClaw loaded the full agent context, the model did not stay completely on the GPU anymore. In my tests it was roughly split between CPU and GPU usage.

For basic tasks this was still useful. For heavier tool usage it was less reliable. The model either needs very concise instructions, or it starts to make mistakes when it has to use tools many times over a longer run.

# Sandbox First

The most important part of the project was sandboxing.

My default idea was that agents should not be able to change files on the host system. Every agent should run in a Docker sandbox, and the default sandbox should have no network access and no writable host workspace.

The rough setup was:

```json
{
  "sandbox": {
    "backend": "docker",
    "mode": "all",
    "scope": "session",
    "workspaceAccess": "none"
  }
}
```

This means that a new Docker based sandbox is created for the agent session. By default, the agent should not be able to freely edit files on the host.

Later I changed the main agent to use `workspaceAccess: "rw"` for testing. The idea was to understand if I could give an agent write access only to its defined workspace and still keep it away from the rest of the host system.

That test was important for my planned use cases. Some agents might need to read or write selected files. But that should happen through a controlled workspace or bind mount, not through full host access.

# What I Tested

I first tried to bind only the default OpenClaw files that looked necessary, like agent instructions and tool files, into the sandbox. My assumption was that I could build a very narrow file view for each agent.

That was not how OpenClaw seems to be designed. The sandbox appears to take the files from the agent workspace at session creation time. They are copied or prepared once for that session, and changing them outside does not automatically change the running container.

When I changed agent files and wanted a clean sandbox again, I used:

```bash
openclaw sandbox recreate --agent webwatch-local
```

This was one of the bigger surprises. The OpenClaw file structure felt more indirect than expected. It is not just "mount this one file and the agent sees the new version". There is a lifecycle around the sandbox and the agent workspace.

# Webwatch Agent

The first real agent idea was `webwatch-local`.

This agent should watch apartment listing websites and store relevant information. Because it needs internet access, it cannot use the default sandbox with no network. For this agent I used Docker networking with `bridge`.

I also tested bind mounts for separate shared data and memory folders. The idea was to give the agent only the folders it really needs, so information can flow between agents:

```json
{
  "sandbox": {
    "docker": {
      "network": "bridge",
      "binds": [
        "workspace/shared/data:/mnt/data:rw",
        "workspace/shared/memory:/mnt/memory:rw"
      ]
    },
    "workspaceAccess": "rw"
  }
}
```

This is the pattern I like more than giving the agent broad access. The agent can write its own data and memory, but it does not need to see the whole server.

# Smart Briefing

I also planned a second agent called `briefing-smart`.

The idea is that the local webwatcher does the repeated collection work. Then the smart briefing agent reads the stored results and creates a short status update for the user. This update can later be sent through a Telegram channel.

For this agent I planned to use a stronger model when needed. The local model is useful for privacy and simple repeated tasks, but a stronger model can be better for summarizing and deciding what is actually important.

# What I Learned

The most important learning was that agent security has to be designed before the agent becomes useful.

It is easy to focus only on the model. But for my use case the sandbox setup was just as important. I needed to know what the agent can read, what it can write, if it has network access, and what survives between sessions.

I also learned that local models can already be useful, but they need small and clear jobs. A local model on edge-like hardware is not the same as a large cloud model. It can work well if the task is narrow, the instructions are concise, and the file access is controlled.

My next step is to make the webwatcher more practical and keep the separation clear: local agents for private repeated work, and stronger models only when the task really needs them.
