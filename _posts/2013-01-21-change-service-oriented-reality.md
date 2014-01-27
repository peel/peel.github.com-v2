---
layout: post-no-feature
title: "Change: The Service Oriented Reality"
description: "Change, impact, effect, reaction. As similar as they might seem some of the concepts revolving around the software change are in fact orthogonal.  
  
The change that drives the business software evolution is twofold. It takes both business and technical change. Both the impact of feature as well as it's maintenance. That is why seemingly orthogonal concepts fit together so well. "
category: articles
tags: [architecture, programming, java]
---

**TL;DR** The article describes the introduction of a pragmatic mini-service architecture. Hints on a distributed software development workflow automation.

# Spike 
Developing an effective, changeable software takes understanding of the nature of change in the context it will be running. Think [Five Ws](http://en.wikipedia.org/wiki/Five_Ws) to be answered when the change occurs. The observation of how it worked in a banking company helped us deliver them an overhauled change-oriented architecture.

The thing about the banking industry is that it fits so well into the domain modeling. The core, supporting and non-domains are easily identified with only a few cross-cutting concerns. It makes it incredibly easy to grow an enterprise system full of pet features, generic solutions and resistant to change. 

With a clear goal in mind and only a bit of domain identification, an observation was made that the real **need** was a limited subset of the core domain services. All the other are unique usecase services.

## The Mini Services

Being pragmatists we wanted to facilitate people's knowledge of the domain where it was crucial. Yet had to avoid too much modeling for the rest. We expected simple and pluggable APIs that encapsulated an **independent part** of a domain. All that in an application small enough one can really [fit in his head](http://vimeo.com/24681032). 
At the time we came up with the idea of something, for the lack of the better name, I call 'mini services'. Something in-between the webserver stack and the [micro services](http://www.infoq.com/presentations/Micro-Services).
The concept of modularization is certainly not a new one. The growing micro-service architectures are just a variation of the [Component-based development (CBD)](http://en.wikipedia.org/wiki/Component-based_software_engineering).   

The micro-services implementation of the CBD assumes full decoupling on both deployment as well as interfacing level. Unlike micro-services we kept our services in a web cluster for sake of keeping the mental model and the familiar tools - see Rich Hickey's [talk](http://www.infoq.com/presentations/Simple-Made-Easy). Still,  the deployments are only a couple of classes in size.  
  
With that said, having an entry point for what we expected to become domains and treating all the rest as non-domains, the solution seemed rather obvious...  

![Mini services: Core Services, Frontend Services, Unique Services](/images/articles/mini-services.png) 

### Core Services
The core services are fundamentally the core domains split into finer-grained, goal-oriented artifacts.
The company provides slightly different business capabilities to its branches in several countries. Having a single services a business concept with several backend representations and minor differences just doesn't cut it anymore. For such cases we needed a single message consuming API that would be able to deliver proper implementation depending on the contents.
A great example of a core service is customer-relationship management API. Each country needs a different holiday of calendar, different data source and representation. Yet aside from data issues the logic stays the same. A simple [content-based routing](http://www.eaipatterns.com/ContentBasedRouter.html) to even deeper service modules solves the problem. And simplifies the deployment.


### Frontend Services
Unless being internal backend services (ie. customer classification services) providing logic to other core services, the core services rarely exist without frontend services. The latter are basically WebAPIs for third parties to interact with the core business concept. They do not contain logic, yet expose just enough core APIs that is needed.  
Front-end services usually provide third parties with REST or SOAP (ekhm, yes, in 2014) APIs.
The drawback of the frontend services is that they cause hidden coupling on deployment level. However the issue is to be simply resolved with event sourcing and message passing.

### Unique Services 
This is probably the most straight-forward part of the platform. These are delivered for a single stakeholder, single usecase and single business problem. With the unique services we can have a full stack of non-shared codebase, data model and interface in a single bundle. Thus, the granularity and simplicity of delivering such services enables us to rewrite a service in a matter of hours. And yes, we did that several times with no harm done.   


# Stabilisation
Few first services were mostly supersimple CRUD data management apps. With just enough thinking to deliver the impacts and fix some of the obvious issues that previously blew up in our faces.  At the time we knew we had to 

> Make things obvious. Break stuff. Ask for feedback.  

After a few deliveries it becomes obvious where the issues are, where's duplication and what needs to be taken care of. Unless you're waterfall/water-scrum-fall The feedback loop should be short enough for you to be fully aware of those in a single release.
  
Now, this is where you roll up your sleeves and make it easy to do good things and hard to do bad, get rid of duplication, make things repeatable, understandable and stable.

## Libraries
We identified that our code either lacked or solved some of the things in a different manner. 

### Logging
Logging, oh sweet, logging. I have never fully understood why people spend hours discussing logging. And above all logging frameworks. And as people tend to be so religious about it and approach... let's take it away from them. And here's where we wrapped logging in several annotations, fluent API and released so everyone can be angry about not using their favourite logging framework anymore.  
After having it for some time it is merely a common idiom even newbies will get. And speaking in idioms is a dream come true.  
Except... unless you're doing this on purpose, for commoditisation of the technology and expressing idioms, don't.

Here's a sample of what we wanted to achieve - standard log format and standard way to log:  
{% highlight java %}
@Log(level=Level.INFO)
public Foo bar(Baz baz){ 
    ...
}
{% endhighlight %}
  
To use the other, more customizable API, you simply make:
{% highlight java %}
public Foo bar(Baz baz){ 
    log.info().message("message {} {}",1,"123");  //logs  INFO   - requestId    | callerId  | userId    |message 1 123
    log.error().requestId("123").message("error"); //logs ERROR - requestId     | callerId  | userId    |error
}
{% endhighlight %}

And still get the standard Ops-friendly format.

### Safety
At the stabilization time we knew that for future's sake we'll need to apply way more sanity checks than we initially put. This is where the safety was born. A library that implements Michael Nygard's [Release it!](http://www.amazon.com/Release-It-Production-Ready-Pragmatic-Programmers/dp/0978739213) concepts. And boy, you'll need one of those as your number of production services and interactions grows. Hopefully Netflix shared a great safety library [Hystrix](https://github.com/Netflix/Hystrix).

Example of safety is a circuit breaker pattern annotation. Each integration point is guarded by a circuit breaker that is triggered after a defined number of exceptions and locked for predefined time:  
{% highlight java %}
@GuardedByCircuitBreaker(exceptionsThreshold=5,retryTimeout=3000) 
public Foo bar(URL url){
    ...
}
{%endhighlight%}

### Monitoring
Monitoring in a heterogenous, distributed environment has a lot of challanges. As we decided to have the services running in a common webserver clusters, the technology the company was using for years, some of the tools have been already available - runtime profiling, request tracking, migration to name a few. However as metrics freaks we needed more. And again it had to be a common idiom. Declarative and transparent. Kind of like [Metrics](http://metrics.codahale.com/) by Coda Hale. Exactly - Metrics. We put some effort to integrate it with our idea of the metrics and monitoring, defined a common concept JSON-based status page holding all the information.   

To get a standard set of metrics we use for each service, you'd simply: 
{%highlight java%}
@DefaultRequestMetrics(id = "Foo") 
public Bar bar(
    Foo parameters) {
    ...
}
{%endhighlight%}
  
Sample status page parsed by monitoring:  
{% highlight json %}
{"version":"3.0.0","gauges":{"FooService.counterGauge":{"value":1},"FooService.heavyCounterGauge":{"value":1001}},"counters":{},"histograms":{},"meters":{},"timers":{}}
{% endhighlight %}

### Template
Before the idea of the service oriented middleware the company had been primarily a Java shop. They've been successfully using Maven for a couple of years, had internal repositories, mirrors, yada yada yada.
Aside from all the [baadddd](http://kent.spillner.org/blog/work/2009/11/14/java-build-tools.html), [bad](http://tech.puredanger.com/2009/01/28/maven-adoption-curve/) vibes maven has, for the straight-forward cases and archetype system it felt the tool to use. The preparation of the uberverbose maven archetype w/ all the modularization we wanted took a bit, yet it was totally worth it. A template with just enough stubbed classes, structure, dependencies set up is a huge value. Just to it.

# Commoditise
The last age of software delivery is commoditisation. The idea of the commoditistion as expressed by [Dan North](http://vimeo.com/43603453) is to further optimise the cost. After having a standard solution to common dilemmas, we had to make it simple to work with the code. That lead us to...

## The distributed development workflow
For a banking company, having a comprehensive service portfolio eventually means hundreds of deployments. This is where the traditional development model fails. Tools fail. Eventually people fail as understanding vanishes. To minimize the impact of high granularity we came up with a simple, yet effective workflow that focuses developers on a single service rather than the full portfolio. This is probably the crown jewel of our platform and the single best reason why it's all working fine to date.

![Distributed workflow](/images/articles/mini-workflow.png)

### This is how we roll
Whenever starting development of a new service you simply create a new Git repo and set it's collaborators. 

Clone it, create a new service out of maven archetype. And at this moment it's ready to be deployed with a single maven command via a dedicated plugin.  

We usually work locally, however at certain point of time you will need to share the service with it's consumers.
Thus to develop a real service you need to create a Jenkins [build pipeline](https://wiki.jenkins-ci.org/display/JENKINS/Build+Flow+Plugin) cloning a defined template:
![Jenkins build pipeline](/images/articles/mini-jenkins.png)  
  
Jenkins' builds are triggered by a webhook whenever a new commit is pushed. Develop builds trigger deployment to early dev environment, we used to call *alpha*.  

When ready to go into testing, you simply execute 'start a new release' in Jenkins. 
The job will branch develop and update versions in Maven poms. After that it builds the artifact that lands as a snapshot in a Nexus binary repository.  

Eventually upon request the CI deploys the artifact to an acceptance environment.

At the time user testing is being made. At certain cases it's also a good practice to mark certain builds as RC. This usually means that the business capabilities are delivered and the changes are 'irrelevant' from business perspective.
Obviously each RC-builds are pushed into Nexus for integration purposes.  

After release decision is made the 'release finish' is executed. This means the release/X.X.X branch is merged into master and Nexus is fed with the release.   
This also marks the moment the generated docs are published into a Service Profile page.  
  
After that the deployment to prelive environment is made. At the moment it would be a real good idea to have a prelive/live routing for subgroup release testing.

### Git
Git was not used at the company before we introduced the approach. However for our purposes [Gitflow](http://nvie.com/posts/a-successful-git-branching-model/) was a match made in heaven. The way it played with the environment of change felt just right. We needed a well-defined flow that would fit company's release cycle compatible approach. We knew Git well enough to share the knowledge with the company's employees. 
Currently each service has it's own repository. Each repository has it's collaborators. People outside of the collaborators group are always welcome to fork  and pull-request the repository. Now that the components are so simple, peer reviews may be done by forking a repository and submitting a pull request.

### Wiki
The great thing about having an [automated](https://bitbucket.org/atlassian/maven-jgitflow-plugin) gitflow is that the CI is capable of pushing the latest, generated docs into company's Confluence. The Confluence contains service profiles describing service metadata (metrics, thresholds), APIs, third party interactions. All the data is generated and pushed into the wiki by Jenkins. Most of the time we simply use an annotation processor for metrics, reaction thresholds etc. APIs however are being documented with [Swagger](http://swagger.wordnik.com/)-compatible [Enunciate](http://enunciate.codehaus.org/) maven plugin. The template usually contains API methods w/ Javadocs, latest API/client maven dependency at times containing samples. 

Of course you could say that all the data is either way available through repository or it's web front. However there are several client systems and service consumers that look for summary about service portfolio and services' capabilities. And for DRY purposes we never edit the description manually. 

### Monitoring
A [monitoring](http://www.nagios.org/) [tool](http://www.zabbix.com/) is being used as an active status pages consumer. It reads JSON pages and pushes notifications according to thresholds set in the service profiles. It is also fed with external data.

One particular thing that we should have had implemented is the 'phone home' pattern. The pattern assumes that each service should **actively** ping back the 'mothership' monitoring tool with a heartbeat. The failure discovery approach along the status pages would have provided enough information on application status. Both Nagios and Zabbix provide a comprehensive APIs for implementing such integration.

Previously I have also mentioned the classic solutions that had existed in the company and needed only a limited effort to get them working for the distributed approach. Each incoming request was marked with an ID that is stored in request header. The ID may be then traced through each service and network component it passes.

### Error Catcher
Having a centralized error catcher ([Sentry](http://getsentry.com/) in this case) enables distributed applications to proactively push each exception to a single WebAPI. The catcher acts as a central storage and dispatcher for issues among applications. As it matches and aggregates exceptions, notifications are distributed according to defined thresholds until fixed (or marked false positive).

# Is it the way to go?
The change context defined the development flow and the architecture. That was arguably the approach to choose when considering service orientation, component-based development and distributed architectures. 


Thus it is extremely important to make the right trade offs. For that as an engineer you should follow what Tim Harford calls the [*Palchinsky Principles*](http://www.amazon.com/Adapt-Success-Always-Starts-Failure/dp/1250007550):
 
> **First:** seek out new ideas and try new things  
> **Second:** when trying something new, do it on a scale that is survivable  
> **Third:** seek out feedback and learn from your mistakes as you go along

Do a few of both core and unique services. Prepare a walking skeleton. Wait till it breaks. Fix it. Do not commit before you measure. Have options. In the exploration you certainly should bite the bullet and do enough experiment to know what seems right for you and what trade offs you will make. 

From the current perspective the only thing I might argue is whether the decision of having services in a single runtime environment was the right tradeoff. It does not overly simplify the deployment nor provides any breakthrough features. On the contrary it does make cross-bundle interaction possible. However the time for the company's developers to pick up the idea, using the familiar tools is now extremely low.

We are now running dozens of services everyone in the development team should be able to fit into their head. The most of the problems are being solved by the outermost line of support. The delivery time is close enough to what we wanted. 
