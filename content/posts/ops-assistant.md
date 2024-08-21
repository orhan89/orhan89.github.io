+++
title = 'Ops Assistant'
date = 2024-08-15T08:47:07+07:00
draft = true
+++

I have been working on a new project I call OpsAssistant over the weekend. It's basically an AI-based assistant to help you resolve IT/Ops problem, but unlike any other AI-based assistant you may have seen out there it won't exactly tell you about the problem or tell you how to resolve it. Instead, it will independently trying to resolve the problem itself with minimal human intervention. 

## Some background

Since the introduction of ChatGPT by OpenAI, people have started using generative AI to help them on their daily work (even this articles are written with the help of ChatGPT). But even ChatGPT or other LLM models alone is pretty limited since all it can do is to generate text after text based on the user prompt. Yes, it can write program code but in the end it still need someone to actually build and run the program. It does not have any means to interract with the outside world. And all this is true until they introduced a new concept called AI Agent.

With Agent, LLM now have a new capabilities to interract with outside world and expand the possibilities that LLM can offer. They allow LLM to access some files/database to look for information, call a code interpreter (like python) to execute code that it write itself, or run some other predefined functions. Learning about this, I wonder if we can finally applied AI to help me to solve my problem as an infrastructure engineer. 

This is basically the idea behind OpsAssistant.

## How it work

![Ops Assistant Architecture](/images/ops-assistant-architecture.svg)

At the heart of this system is LLM model itself. There are many LLM model that has been developed out there, but so far i have only tried using OpenAI GPT 4o. Next I might want to try using difference models.

I use a framework called LangChain to help me build the AI capability for this project. LangChain offer an easy way create `chains`, which are sequences of actions that language model will take. This include tasks like information retrieval, decision-making, or interacting with external entities through tools/APIs, enabling a more complex workflows. As an assistant to investigate and resolve IT infrastructure, the LLM model would need to access several tools that is common to IT operation tasks to investigate and resolving IT problem like local shell, remote shell (ssh), kubectl, or any other tools. LangChain also abstract the languages model and make it easier for us to change the LLM model to use. 

The original implementation of LangChain is in python, but at the time i write this article there are already many alternative implementation written in several languages. This project itself are using the elixir implementation, basically because it's designed to be used in functional style, and I would like to use Phoenix and LiveView to build the webhook endpoint and interractive web app part later.

The system will receive new alerts through webhook. This allow us to integrate OpsAssistant with any existing monitoring/alerting system like prometheus/alertmanager, pagerduty, opsgenie, or event slack. For every incoming alert from those system, OpsAssistant will run the LLM model using the information from the alert to investigate and resolve the problem using available tools. For now i have only implemented `local_shell` that allow the LLM to execute a command on local shell, but next I am planning to implement tool to execute command in remote shell (through ssh) and some other tools as well.

If you raised in the 90's or have watched Terminator, you will know that we can't totally trust the AI, especially when they can interract with the outside world. For instance, as an infrastructure engineer you shouldn't let AI to call a command in your shell without your supervision (you definitely don't want the AI to run rm -rf /). Fortunately, LangChain had the option to stop the chain right before executing the tools. This give us a means to keep human in control by implementing an approval process in place. I decide to go with Phoenix and LiveView for the web app part due to the high interractivity nature of this processes.

## Results

To illustrate how it works, I used a tool called `stress` to emulate High CPU Usage  on a machine where OpsAssistant is run. It can be seen from the `htop` below the command that all 8 cpus is being 100% used shortly after I run `stress`

![Run stress tool](/images/stress-1.png)

My monitoring tool will then reporting this alert to OpsAssistant and forward it to the LLM model. 

![Calling top](/images/ops-assistant-4.png)

It can be seen from the response above, the LLM try "to gather data on current CPU usage and identify processes consuming high CPU" by calling "local_shell" tool to run `top -b -n1 | head -n20` command. 

Someone would then need to approve (or declined) this call before the tool can execute the command and return the results back to the LLM model.

![Result from top](/images/ops-assistant-5.png)

![Result from top](/images/ops-assistant-6.png)

From the tool response we now know that the CPU is used mostly by the `stress` processes (there are 8 of them). We can then ask the LLM to continue the investigation.

![Result from top](/images/ops-assistant-8.png)

Upon further run, the LLM model is then responding with another tool call series to kill each `stress` processes that is using all the CPU. Of courcse I don't mind to let it execute them this time because I know that's what I would do myself.

![Result from top](/images/ops-assistant-10.png)

Once the tool finished and return the response back, we can see that only one call is succeed. This is expected as only the main process need to be terminated, which will terminate all the child processes. 

We can then ask the LLM to continue once again having this new information.

![Result from top](/images/ops-assistant-12.png)

As expected, the LLM decide to call `top` command again to "To verify if CPU usage has decreased after terminating the stress processes", and after approve the call and tool returned with now a lower CPU usage, it response by saying that "The high CPU usage issue has been resolved. CPU usage has significantly decreased after terminating the `stress` processes."
