---
title: 'An AI based assistant for Infrastructure Engineer'
date: 2024-08-15T08:47:07+07:00
omit_header_text: true
draft: true
featured_image: '/images/ops-assistant-banner.jpg'
---

I have been working on a new project called *OpsAssistant* over the weekend. It's an AI-based assistant to help you resolve IT/Ops problems, but unlike any other AI-based assistant you may have seen out there it won't exactly tell you about the issue or tell you how to resolve it. Instead, it will independently try to fix the problem with minimal human intervention. 

## Why Now?

Since the introduction of [ChatGPT by OpenAI](https://openai.com/index/chatgpt/), people have started using generative AI to help them with their daily work (even these articles are written with the help of *ChatGPT*). But even *ChatGPT* or other [Large Language Models](https://en.wikipedia.org/wiki/Large_language_model) alone are pretty limited since all it can do is generate text after text based on the user prompt. Yes, it can write program code but in the end, it still needs someone to build and run the program. It had no any means to interact with the outside world. And all this was true until they introduced a new concept called [AI Agent](https://github.blog/ai-and-ml/generative-ai/what-are-ai-agents-and-why-do-they-matter/).

With *Agent*, LLM now has new capabilities to interact with the outside world and expand the possibilities that LLM can offer. They allow LLM to access some files or databases to look for particular information, call a code interpreter (eg. *python*) to execute code that it writes itself, or run some other predefined functions. Learning about this, I wonder if we can finally apply AI to help me solve my problem as an infrastructure engineer. 

All this is the idea behind *OpsAssistant*.

## Now tell me how it works!

![Ops Assistant Architecture](/images/ops-assistant-architecture.svg)

At the heart of this system is the *LLM* itself. Many LLMs have been developed and published out there, but so far I have only tried using OpenAI *GPT 4o* for this project. (Notes: I might want to try using different models and compare the results in the future)

I use an [elixir](https://elixir-lang.org/) adaptation of an AI framework called [LangChain](https://www.langchain.com/) to help me build the AI capability for this project. *LangChain* (and all of its adaptations) offers an easy way to create [_chains_](https://python.langchain.com/v0.1/docs/modules/chains/) (sequences of actions that the language model will take). This includes tasks like information retrieval, decision-making, or interacting with external entities through tools/APIs, enabling more complex workflows. *LangChain* also abstracts the language model and makes it easier for us to change the model to use. 

The [original implementation](https://github.com/langchain-ai/langchain) of *LangChain* is in [*python*](https://www.python.org/), but at the time I wrote this article, there were already many alternative implementations/adaptations written in several languages. This project itself uses the [*elixir* adaptation](https://github.com/brainlid/langchain), basically because it's designed in a functional style, and I would like to make it work with [*Phoenix*](https://www.phoenixframework.org/) and [*LiveView*](https://github.com/phoenixframework/phoenix_live_view) to add the interactivity part later.

The system will receive new alerts through webhook. This allows us to integrate OpsAssistant with any existing monitoring/alerting system like *Prometheus/Alertmanager* or incident management platforms like *Pagerduty*, *Opsgenie*, or team communication platforms like *Slack*. For every new incoming alert from those systems, *OpsAssistant* will run the *LLM* using the information from the alert to investigate and then resolve the problem using available tools. As an assistant to investigate and resolve IT infrastructure, the LLM model would need to access several tools that are common to IT operation tasks to investigate and resolve IT problems. For now, I have only implemented _local_shell_ tool that allows the LLM to execute a command on the local shell, but I have been thinking of implementing another tool to execute a command in the remote shell (through *ssh*) and probably some other tools as well.

If you were raised in the 90s or have watched [*The Terminator*](https://www.imdb.com/title/tt0088247/), you would understand that we can't trust the AI especially when they can interact with the outside world. For instance, as an infrastructure engineer you shouldn't let AI call a command in your shell without your supervision (you definitely don't want the AI to run "*rm -rf /*"). Fortunately, *LangChain* had the option to stop the chain right before executing the tools. By implementing an approval process in place, we ensure that humans are still in control. I decide to go with *Phoenix* and *LiveView* to implement this capability due to the high interractivity nature of this processes.

## Time for some tests

To illustrate how *OpsAssistant* works in action, I used a tool called [stress](https://github.com/resurrecting-open-source-projects/stress) to emulate high [CPU load](https://en.wikipedia.org/wiki/Load_(computing)) on the machine where *OpsAssistant* is running. I can check from CPU monitoring tools like [htop](https://htop.dev/) that all my CPU is now being 100% used shortly after *stress* tool is running.

![Run stress tool](/images/stress-1.png)

In the original idea, a monitoring tool would then report this alert to *OpsAssistant* before forward it to the *LLM*. But for now, I will report the alert manually by creating a new alert in *OpsAssistant*. 

![Calling top](/images/ops-assistant-4.png)

It can be seen from the screenshot above that the *LLM* respond to the alert by requesting the *local_shell* tool to run "*top -b -n1 | head -n20*" command *to gather data on current CPU usage and identify processes consuming high CPU*. 

I would need to approve this call before it can be executed and return the *results* to the LLM.

![Result from top](/images/ops-assistant-5.png)

![Result from top](/images/ops-assistant-6.png)

From the tool call response, we (and the *LLM*) now know that the CPU is mostly used by the *stress* processes (there are 8 of them). I asked the LLM to continue the investigation then.

![Result from top](/images/ops-assistant-8.png)

Upon further run, the LLM then requests the *local_shell* tool to run a series of *kill* commands *to terminate the stress process consuming high CPU*. Of course, I don't mind letting it execute them this time because I know that's what I would do myself in this situation.

![Result from top](/images/ops-assistant-10.png)

Once the tool is finished and response is returned, we can see that only one call has succeeded. This is expected as only the main process needs to be terminated, which will terminate all the child processes. 

I then ask the LLM to continue once again.

![Result from top](/images/ops-assistant-12.png)

As expected, the LLM requested to execute the *"top -b -n1 | head -n20"* command again *to verify if CPU usage has decreased after terminating the stress processes*, and after approving the call and the tool returned with now a lower CPU usage, it responded by saying *"The high CPU usage issue has been resolved. CPU usage has significantly decreased after terminating the "stress" processes."*

## Conclusion

It may be too soon to conclude whether the solution that I proposed here with *OpsAssistant* is good enough to resolve IT issues in general, but the results look promising so far. The new solution seems to be able to investigate and resolve the given "High CPU Load" alert independently while I can sit still and only need to review any requested tool call. I still need to perform some more testing with different alerts before I would consider it to be used in production, and probably compare between available AI models to get the best results.
