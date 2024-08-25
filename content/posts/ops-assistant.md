---
title: 'An AI based assistant for Infrastructure Engineer'
date: 2024-08-15T08:47:07+07:00
omit_header_text: true
draft: true
featured_image: '/images/ops-assistant-banner.jpg'
---

I have been working on a new project I call *OpsAssistant* over the weekend. It's basically an AI-based assistant to help you resolve IT/Ops problem, but unlike any other AI-based assistant you may have seen out there it won't exactly tell you about the problem or tell you how to resolve it. Instead, it will independently trying to resolve the problem itself with minimal human intervention. 

## Why even bother?

Since the introduction of [ChatGPT by OpenAI](https://openai.com/chatgpt/), people have started using generative AI to help them on their daily work (even this articles are written with the help of *ChatGPT*). But even *ChatGPT* or other LLM models alone is pretty limited since all it can do is to generate text after text based on the user prompt. Yes, it can write program code but in the end it still need someone to actually build and run the program. It does not have any means to interract with the outside world. And all this is true until they introduced a new concept called [AI Agent](https://github.blog/ai-and-ml/generative-ai/what-are-ai-agents-and-why-do-they-matter/).

With *Agent*, LLM now have a new capabilities to interract with outside world and expand the possibilities that LLM can offer. They allow LLM to access some files or database to look for particural information, call a code interpreter (eg. *python*) to execute code that it write itself, or run some other predefined functions. Learning about this, I wonder if we can finally applied AI to help me to solve my problem as an infrastructure engineer. 

This is basically the idea behind *OpsAssistant*.

## How it work

![Ops Assistant Architecture](/images/ops-assistant-architecture.svg)

At the heart of this system is [Large Language Model (LLM)](https://en.wikipedia.org/wiki/Large_language_model) itself. There are many LLM that has been developed and published out there, but so far I have only tried using OpenAI *GPT 4o* for this project. (Notes: I might want to try using difference models and compare the results in the future)

I use an adaptation of a framework called [LangChain](https://www.langchain.com/) to help me build the AI capability for this project. *LangChain* offer an easy way create _chains_, sequences of actions that language model will take. This include tasks like information retrieval, decision-making, or interacting with external entities through tools/APIs, enabling a more complex workflows. LangChain also abstract the languages model and make it easier for us to change the *LLM model* to use. 

The original implementation of *LangChain* is [in *python*](https://github.com/langchain-ai/langchain), but at the time I write this article there are already many alternative implementation written in several languages. This project itself are using the [*elixir* implementation](https://github.com/brainlid/langchain), basically because it's designed in functional style, and I would like to made it work with [*Phoenix*](https://www.phoenixframework.org/) (and *LiveView*) to add interractivity part later.

The system will receive new alerts through webhook. This allow us to integrate OpsAssistant with any existing monitoring/alerting system like [*prometheus/alertmanager*](https://prometheus.io/) or incident management platform like [*pagerduty*](https://www.pagerduty.com/), [*opsgenie*](https://www.atlassian.com/software/opsgenie), or team communication platform like [*slack*](https://slack.com/). For every new incoming alert from those system, *OpsAssistant* will run the *LLM* using the information from the alert to investigate and then resolve the problem using available tools. As an assistant to investigate and resolve IT infrastructure, the LLM model would need to access several tools that is common to IT operation tasks to investigate and resolving IT problem, eg. *local shell*, *remote shell* (*ssh*), *kubectl*, or any other tools. For now I have only implemented _local_shell_ tool that allow the LLM to execute a command on local shell, but I have been thinking to implement another tool to execute command in remote shell (through *ssh*) and probably some other tools as well.

If you raised in the 90's or have watched *Terminator*, you would understand that we can't totally trust the AI especially when they can interract with the outside world. For instance, as an infrastructure engineer you shouldn't let AI to call a command in your shell without your supervision (you definitely don't want the AI to run rm -rf /). Fortunately, *LangChain* had the option to stop the chain right before executing the tools. By implementing an approval process in place, we ensure that human are still in control. I decide to go with *Phoenix* and *LiveView* to implement this capability due to the high interractivity nature of this processes.

## It's time for some test

To illustrate how *OpsAssistant* works in action, I used a tool called [stress](https://github.com/resurrecting-open-source-projects/stress) to emulate *High CPU Load* on the machine where *OpsAssistant* is running. I can check from cpu monitoring tool like [htop](https://htop.dev/) that all my cpus is now being 100% used shortly after *stress* is running.

![Run stress tool](/images/stress-1.png)

In the original idea, monitoring tool will then reporting this alert to *OpsAssistant* and forward it to the LLM model, but for now i will report the alert manually by creating a new alert. 

![Calling top](/images/ops-assistant-4.png)

It can be seen from the screenshot above, that the LLM model will response the alert by requesting *local_shell* tool to run "**top -b -n1 | head -n20**" command *to gather data on current CPU usage and identify processes consuming high CPU*. 

I would need to approve this call before it can be executed and return the *results* back to the LLM model.

![Result from top](/images/ops-assistant-5.png)

![Result from top](/images/ops-assistant-6.png)

From the tool call response we (and the LLM model) now know that the CPU is mostly used by the *stress* processes (there are 8 of them). I asked the LLM to continue the investigation then.

![Result from top](/images/ops-assistant-8.png)

Upon further run, the LLM model is then requesting *local_shell* tool to run a series of *kill* command *to terminate the stress process consuming high CPU*. Of course I don't mind to let it it execute them this time because I know that's what I would do myself in this situation.

![Result from top](/images/ops-assistant-10.png)

Once the tool finished and return the response back, we can see that only one call is succeed. This is expected as only the main process need to be terminated, which will terminate all the child processes. 

I then ask the LLM to continue once again.

![Result from top](/images/ops-assistant-12.png)

As expected, the LLM requesting to execute `top` command again to *to verify if CPU usage has decreased after terminating the stress processes*, and after approve the call and tool returned with now a lower CPU usage, it response by saying that "The high CPU usage issue has been resolved. CPU usage has significantly decreased after terminating the `stress` processes."

## Conclusion

It may be too soon to conclude either the solution that I proposed here with OpsAssistant is good enough to resolve IT issues in general, but the results are looks promising so far. The *OpsAssistant* is seems to be able to investigate and resolve the given High CPU Load alert independently while I can sit still and only need to review any requested tool call. I still need to perform some more testing with different alert before I would consider it to be used in production, and probably compare between available AI models to get the best results.
