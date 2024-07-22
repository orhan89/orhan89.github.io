+++
title = 'Maximizing Value as an Infrastructure Engineer'
date = 2024-07-20T01:31:32+07:00
draft = false
+++

As an Infrastructure Engineer (or DevOps/Cloud/Platform engineer, you name it), your work can be grouped into three types:

- Incident/Alert Handling
- Maintenance
- Projects

Of these three types, only one actually generates values. Unfortunately, most of our time are likely to be spent on the other two types that aren't. Hence, the question of how to maximize your values as an engineer is simply all about refocusing your work from something that aren't generating value to something that are. In the subsequent section, i will talk about some approaches that I have taken to achieve that goal.

# Incident Handling

Incident handling perhaps is the type of work for an infrastructure engineer that gets the most visibility from the others. We used to judge an engineer by their ability to solve incidents. Even many times, an engineer got labeled as a hero after successfully resolving one. While this is not exactly false, there are other ways to see it.

The thing is, **resolving an incident is not actually generating value**. For what it's worth, it is merely preventing you or your company from further loss, and it is not the same thing as generating value. If the further loss is bad, then why don't we try to not allow the incident in the first place? Hence, we really shouldn't judge an engineer on their ability to resolve an incident alone, but also on **how far we can go to prevent the incident by improving the reliability of the system**. I will talk in more detail about this in the subsequent section.

Still, when an incident happens all of our effort should be put into resolving it as soon as possible, since the total loss that you will suffer is proportional to the length of the incident. In the infrastructure world, this is usually measured in a metric called _Mean Time To Repair (MTTR)_ - the average time it takes you to resolve every incident during some period.

Now there are many ways to solve an incident faster, one of them is to have some documentation or guidelines of what to do when a type of incident happens (or in the infrastructure world people usually call this a _runbook_). But just having a runbook doesn't free you from the physical limitation of having to perform each action by hand. This is where **automation** can help you further. At NiceDay we have an internal tooling called _Regat_ that contains a collection of investigation/mitigation actions that you can perform when an incident happens with just one click, fully automatic.

However, resolving incidents quickly shouldn't be the only goal when it comes to incident handling. You may only need 5 minutes to resolve the incident, but if you need 15 minutes before you are aware of the incident, then it will still count as 20 minutes as the resolution time. This metric is usually measured as _Mean Time To Acknowledge (MTTA)_ or the average time for an incident to be detected and acknowledged by someone. To ensure that you are immediately aware of any possible incident, you need to have a good **monitoring** system, that is sensitive enough to detect the smallest issues, but accurate enough to filter out as many false alarms as possible. There are many options to monitor your system out there, but I recommend using _Prometheus_.

One big problem with incident handling is that, unlike the other two types of work, it is so unpredictable that it can happen at any time. That means you need to be always ready and sprightful so you can handle an incident immediately. It will also take all your focus when it's happened. This requires a lot of your mental energy, and if it's not carefully taken care of it can prevent you from having any other activities or work. This is why it's so important to implement an **on-call schedule** so the mental load can be distributed between the team.

Furthermore, there is one more metric called _Mean Time Between Failure (MTBF)_ that you should be aware of, that is the average time it needs for an incident to happen again in the future. An incident that takes you 10 minutes to resolve but happens every week for a month is equal to 40 minutes of incident. This problem can be solved by the other type of work that I will explain in the next section.

# Maintenance

The next type of work that an engineer can perform is maintenance. Unlike incident handling, maintenance work is more predictable and less mentally draining. But just like incident handling, **maintenance doesn't generate much value (if not at all)**.

To understand why maintenance is still important, one needs to know that many types of incidents are like cracks in some machines. After some time, the cracks are getting bigger or piling up, and at some point, they will try to break loose. To prevent that, we can either patch or replace the compromised component so the machine can last longer. Doing this perhaps will prevent you from having the mental turmoil of handling a bad incident.

At the start, it may sound like a good idea to have a collection of your inventory along with all of their maintenance item and schedule. By following the schedule, you can have a chance to avoid some incidents from happening. After a while, you will have fewer incidents in your system, hence a better MTBF. However, performing maintenance alone may not eradicate incidents completely. And when your system has become bigger and more complex, your maintenance item is also become bigger. At some point, it will take all the time left you have. And this is where your plan needs to be upgraded. 

The first thing that you can do at this point is to **automate the maintenance** itself. Instead of manually upgrading a VM or the operating system, you may want _Terraform_ to do that for you. Instead of periodically cleaning up your test data by hand, you may want to let _Ansible_ do that for you (or maybe just a cronjob would be enough). Those actions would reduce the time you need for each maintenance item.

And if that alone is not enough, perhaps you want to offload that load somewhere else. I have to admit that we are living in a different world now than 10-20 years ago, with all this _SaaS (Software as a Service)_ available to use. You can now "order" a new PostgreSQL instance in _Google Cloud Platform CloudSQL_ and let _Google_ do most of the maintenance. It may not always meet your requirement or use case, but if it is, and if the cost is still justifiable, I would recommend you to at least considering to use them because doing so can save you a lot of time from doing maintenance yourself. 

We usually call this metric as **maintainability**, that measures the time that you need in maintenance to be effective enough in preventing incidents. As an engineer, we should always try to improve the maintainability of our system by either reducing the time it needs for maintenance or reducing the number of components that we need to maintain by reducing the complexity of the system. And with the precious time that you have now, you can start focusing on the next type of work, the one that in the beginning I had mentioned to be the only one that generates value.

# Projects

Now we have come to the last type of work for an engineer. It could be an ordinary project where our customers can directly get benefit from the results, like developing a new feature of an existing product or even a new product itself. It could also be an internal project where the results can be used by the company or other teams in the organization.

As I mentioned in the beginning, **projects are the only type of work that could generate value**. But even projects are only generating value when they finally reach their designated users. That means to start generating value, we need to first deliver them. And because the time spent without delivering is times where we are not generating value as well, we need to **deliver them fast**. 

The ability to deliver fast in a complex system used to be seems impossible not so long time ago. Fortunately, we have seen a lot of progress and new technology that try to resolve this lately. From my own experience, I feel technology like _Kubernetes_ has lifted most of the work needed to deploy a new complex system by abstracting away the common and low-level components. Introducing a _Continuous Delivery (CD)_ system will simplify the deployment process even further by requiring only a push of a new commit to the git repo.

Capabilities to deliver fast also have another side effect: it allows us to **deliver more often**. And this means we generate way more value than before. In the end, I believe the ability to deliver fast and often is the key to a productive and impactful team. Just for illustration, at NiceDay we normally deliver around 70 changes in just a single day. 

Values can also be generated by helping the company (or any part of it, including yourself) to create further values, and this should be the idea of having **internal projects**. In the previous section, I mentioned how we can reduce the number of incidents by modifying the existing system to be more reliable; this is a project on its own. Implementing a new system so our developer can easily deploy a new test and development environment on their own has boosted productivity by a degree because now they don't have to deal with the long setup process anymore. Developing a tool and system to streamline developers' requests has also broken the wall between the developer and ops team and made the process much faster. Perhaps I can talk more about this in the future, but for now, I can say with confidence that building your internal tooling will benefit you more than you think.

Another great example of internal projects is **cost optimization** because the cost that you save will become an increase in revenue by the end of the day.

# Conclusion

So far I have talked at some great length about three types of work an infrastructure engineer can do. It seems like each type is standing on its own, but in fact it's interrelated because unlike computers you can only do one work at a time. When all of your time is consumed to handle incidents and maintenance, you might feel like there's no time left to do projects. In this case, it might good idea to postpone some or all the maintenance and start working on an internal project to improve reliability. Start by doing a post-mortem analysis of the past incidents and try to look for the pattern. After finding the weakest component, you may want to try to fix it or even replace it with a better one. If you are lucky, you may have fewer incidents coming. 

Now that you have some handy spare time, you might think to finish all the maintenance that you have put on pending for a long, but I would suggest moving your commitment to another project to improve the maintainability for at least half of the available time. By the end, you will find that you now finally have time that you probably have never imagined before, time that you can use to actually generate values.
