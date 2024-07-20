+++
title = 'Maximizing Value as Infrastructure Engineer'
date = 2024-07-20T01:31:32+07:00
draft = true
+++

As an Infrastructure Engineer (or DevOps/Cloud/Platform engineer), basically your work can be grouped into three types:

- Incident/Alert Handling
- Maintenance
- Projects

Of these three types, only one actually are generating values. Unfortunately, most of our time are likely to be spent on the other two types that aren't. Hence, the question of how to maximizing your values as engineer are simply all about refocusing your work from something that aren't generating value to something that are. In the subsequent section, i will talk about some approaches that I have taken to achieve that goal.

# Incident Handling

incident handling perhaps is the one most seen work, happened all the time. many times, an engineer value is seen from being hero when succesfully resolved an incident.

The thing is, resolving an incident is not actually generating value. for what its worth, it is merely preventing you or your company from loss. It is not a same thing as generating value. hence, we shouldn't judge an engineer from ability to resolve incident alone.

Unfortunately, loss is not exactly something that you can just ignore. When an incident happened, all of our effort should be put to resolve it as soon as possible. The loss that you derita is proportional to the length of incident. In infrastructure world, this usually measure in a metric called MTTR (Mean Time To Resolve) - the average times it takes you to resolve for every incident during some period.

Now there are many way to solve an incident faster, and of them are to have some documentation or guideline of what to do when a type of incident is happened (or in infrastructure world people usually called this a runbook). But just having a runbook doesn't free you from physical limitation of having to perform each action by hand. This is where automation can help you further. At NiceDay we have an internal tooling call Regat that contain a collections of investigation/mitigation action that you can perform when an incident happened with just with one click, fully automatic.

But resolving incidents quickly shouldn't be the only goal when it comes to incident handling. You may only need 5 minute to resolved the incident, but if you need 15 minute to be aware of the incident, than the incident is still count to be 20 minutes. This metrics is usually measured as MTTA (Mean Time To Acknowledge) or the average time for an incident to be detected and acknowledge by someone. To ensure that you immediately aware for any possible incident, you need to have a good monitoring system, that is sensitive enough to detect the smallest issues, but accurate enough to filter out as much false alarm as possible. There are many option to monitor your system out there, but i personally recommend to use prometheus.

One big problem with incident handling, is that because unlike the other two type of work, it is so unpredictable that it can happend any time. That mean you need to be always ready and sprightful so you can handle an incident immediately. It will also takes all your focus when it's happened. This require a a lot of your mental energy, and if it's not carefully taken care of it can prevent you from having any other activities or work. This is why it's so important to implement on-call schedule, so the mental load can be distributed between the team.

Furthermore, there is one other metrics called MTBF that you should be aware of, that is the average time it need for an incident to happen again in the future. An incident that take you 10 minute to resolve but happened everyweek for a month is equal to 40 minutes of incident. This problem can be solved by the other type of work that i will talk in the next section.

# Maintenance

The next type of work that an engineer can perform are maintenance. Unlike incident handling, maintenance work are more predictable, and less mentally draining. But just like incident handling, it doesn't generate much value (if not at all).

To understand why maintenance is still important, one need to know that many types of incident is like a time bomb, that keep ticking until it went off. Or you can also see it as a crack in some machine. By the time, the cracks is getting bigger and piling up, and at some point it will try to break loose. To prevent that, we can either patch or replace the component with the crack, so the machine can last longer. This will prevent you from having the mental turmoil of handling a bad incident.

As a start, it maybe good idea to have a collection of your inventory along with all of their maintenance item and schedule. By following the schedule, you can have a chance to avoid some incident from happening. After a while perhaps now you have less incident in your system, hence a better MTBF. But performing maintenance alone may not eradicate incident completely. And when your system is become bigger and complex, your maintenance item is also become bigger. At some point it will take all the time left you have.

This is where your plan need to be upgraded, and the first thing that you can do from here is to automating the maintenance itself. Instead of manually upgrading a VM or the operating system, you may want Terraform to do that for you. And instead of periodically cleanup your test data by hand, you may want Ansible to do that for you (or maybe just cronjob would be enough). Those action would reduce the time you need for each maintenance item.

And if that alone is not enough, perhaps you want to offload that load to someone. I have to admit that we are living in a different world now than 10-20 year ago, with all these Software as a Service. You can now "order" a new PostgreSQL instance in Google Cloud Platform CloudSQL and let them do the most of the maintenance. It may not always met your requirement or use case, but if it is, and if the cost is still justifiable, I would recommend you to atleast considering to use them because by doing so can save you a lot of time from doing maintenance yourself. And having this precious time, you can start focus on the next type of work. The one that in the beginning is way more important.

At NiceDay we call this maintainability, or the time you need to do maintenance to be effective enough in preventing incidents. And we thrive to always improve the maintainability by either reducing time it need for maintenance, or reducing the number of component that we need to maintain by reducing the complexity of the system.


# Projects

Now we have come to the last type of work for an engineer. It could be an ordinary project that the results can directly be used by our customer, like developing a new feature of existing product or even a new product itself. It could also be an internal project that the results can be used by company or other team in the organization.

As i mentioned in the beginning, projects are the only type of work that could generate values. But even projects are only generating value when it finally reach their designated users. Thats mean to start generating value, we need to first deliver them. And because the time spent without delivering is times where we are not generating value as well, we need to deliver them fast. 

The abiliy to deliver fast in a complex system used to be seems impossible not so long time ago, but fortunately we have seen a lot of progress and new technology that try to resolve this lately. From my own experience, I have feel technology like kubernetes (and helm) have lifted most of the work needed to deploy a new complex system by abstracting away the common and low level component. And introducing CI/CD system even simplifying the deployment process even further by just pushing a new commit of the last change to git repo.

Capabilities to deliver fast also has another side effect: it allow us to deliver more often. And this means we generate way more value than before. In the end, we believe the ability to deliver fast and often is the key to be a productive and impactful team. Just for illustration, at NiceDay we normally deliver around xx changes in just a single day. 

Values can also be generated by helping company (or another part of it) to generate further values, and this should be the idea of having internal projects. Implementing a new system so our developer can deploy a new test and development environment on their own easily had boost their productivity by a degree because now they don't have to deal with the long setup process anymore. Developing a tool and system to help developer to request some changes to infrastructure has also broke the wall between developer and ops team and making the process more streamlined and fast. Perhaps I can talk more about this in the future, but for now I can say with confidence that building your own internal tooling will benefit you more often than you think.

Another good example of this internal projects are cost optimization, because the cost that we saved by the end of the day will become increase in the revenue.


# Conclusion


So far i have talk at some great length about three types of work an infrastructure engineer can do. It seems like each type is standing on their own, but in fact its saling berhubungan because unlike computer you can only do one work at a time. When all of your time is consumed to handle incidents and basic maintenance, you might feel like there's no time left to do projects. In this case, it might good idea to postponed some or all the maintenance and start working on an internal project to actively reduce the number of incidents. Start by doing a post-mortem analysis on the past incidents and try to look for the pattern. After finding the weakest component, you may want to try to fix it or even replace it with a better one. If you are lucky, you may have less incidents coming. And now that you have some handy spare time you might think to finish all your maintenance that you have put on pending for long, but I would suggest to keep your commitment for another projects to improve the maintainability for at least half of the available time. By the end, you will find that you now finally have time that you probably have never imagined before, time that you can use to actually generate values.
