---
layout: post
title:   Systems Design - Understanding monolith and microservice patterns
categories: [SRE, High Availability, Microservices, System Design ]
excerpt: Design a weather web application to understand monolith design vs microservice design pattern 
---
Microservices are a collection of services that work together to serve an application’s purpose. Contrast this with a monolith service that includes all functions and features in one application. 

Most simple applications will start out as a monolith. For example, a  simple helloworld.py file  that prints ”hello world” is a monolith.  

Big tech companies operate at massive scale, in terms of data volume, user traffic or number of engineers building the applications. This is why microservices are the preferred architecture pattern in most large scale production systems. 

Let’s try and understand each architecture better by building an example application. 

**Objective: Build a simple web application that greets the visitor and displays the current weather of the visitor’s location.**


## Monolith Architecture Weather App

If we were to build this as a monolith, we might create a python flask application that accepts the HTTP Request, and get the visitor’s IP address from the browser. 

Then a function within the application converts the visitor’s IP address to a city name or co-ordinates.  

Next the application makes a separate call to one of the many weather data providers with the given city name as a query, and then returns the temperature to the user.  

All this happens under one running application, which is called the monolith. This application is then deployed in a single server in our hosting provider or datacenter. 


![alt_text](images/weather-monolith.png "image_tooltip")


Fig 1: Monolith weather application 

Now let's identify some pros and cons with this monolith architecture.

Pros:



* Simple Deployment - It is deployed on a single application instance/host. 
* Simple infrastructure - No need for complex concepts such as containers, load balancers, service discovery etc. 
* Easy to troubleshoot - If the weather API is down, or the server has memory issues etc, it is easy and straightforward to debug. The logs are all in the one server. 

Cons:



* There is no redundancy - If the application goes down, there is no backup. 
* It is difficult to scale - If the user base increases to hundreds of thousands, then the single application will struggle to handle the load.
* Difficult codebase for multiple developers - If we want to expand with features such as adding databases, adding advertisement, more complex business logic, then multiple developers will be working on the same codebase, which can get messy real fast. 
* Delayed features - If new features need to be built, it could be delayed since the deployment depends on other parts of the application needing change. 


## Microservices Architecture Weather App

Now lets try building the same application as a microservice. 

We start by  building a web/frontend that will accept the user request and serve data back to the user. We can use a simple python flask application for this, we’ll call it the weather-frontend-service.  

Next we’ll create a new service called geo-ip-service. This application will convert the IP address received from the frontend-service, and convert it into a city name or co-ordinates. It does this by looking up the IP address in a local geo-ip-database file. 

Next we’ll create a new service called weather-lookup-service, that will make a call to the weather data providers.  The sole job of this weather-service application is to make calls to the external weather data API. 

Now when the user visits the website, the frontend gets the client’s IP address and passes that to the geo IP data lookup service, which will return the city name.  The frontend will then make a call to the weather-lookup-service application to get the weather data. The weather-lookup-service does its job and returns the weather info to the frontend. The frontend then serves the current temperature to the visitor. 

Here’s a simple implementation of this basic client-server microservice using python and grpc. 

[https://github.com/tgangte/python-grpc-weather-microservice](https://github.com/tgangte/python-grpc-weather-microservice) 



![alt_text](images/weather-microservice.png "image_tooltip")


Fig 2: Microservices weather application 

These individual applications can then be containerized, that is deployed individually in virtual containers such as docker. We can also run them as individual pods in Kubernetes, for example. These pods then communicate with each other with kubernetes services and they can be scaled up or down with deployments. 


### How can we improve this system design further? 

Let’s say we have rapid growth and receive 1000 simultaneous visitors from Berlin, Germany, that means 1000 API calls are made to the weather data provider, and each call costs money. We might also be rate limited to a certain request per second. 

Since the weather in Germany is approximately the same for all users, we can introduce a cache such as Memcached or Redis to temporarily store the weather data for every requested city for say 10 minutes. Now all 1000 requests will be served from the cache and the external API call will only be used to make fresh calls every 10 minutes, drastically reducing the API subscription cost or request rate.  

Now let's identify some pros and cons with this microservice architecture.

Pros:


* Reliability - If any service goes down, we can have redundancies to take over the workload. 
* Scalable - The website is now horizontally scalable. We can add more frontends, more geo-ip-data services that can run independently and scale up to meet demand. 
* Rapid and independent deployment - Since each function of the website is now a different service, they can be worked on by different teams or engineers independently. They can also be independently deployed without affecting or being dependent on the other services. 

    For example, if there is a need to change weather API providers, we can make the change in the weather-lookup-service. The other services do need to know about this change or be impacted by it, since the lookup service returns the same data as before. 


Cons:


* The application is now much more complex: 
    * Each microservice communicates to the other using network calls. 
    * Inter-services communications need to happen over REST, GRPC, etc
* Expensive - It is more expensive to build and maintain, due to engineering skills and infrastructure costs
* It is more difficult to troubleshoot when things go bad, since there are so many moving parts. 
* Introduces the need for more advanced infrastructure such as containers, service discovery, messaging queues, load balancers and distributed tracing and logging.


## Conclusion - Which is better?  Which one should we use?  


There is no one size fits all solution, but small companies with stable traffic and data volumes should generally avoid the complexities of microservices architecture. Simpler approaches often work best for smaller operations.

Large companies or companies that anticipate rapid growth and scaling typically design their infrastructures ground up as microservices. This is of course accompanied by increased costs as more advanced skill sets such as SRE and DevOps are needed along with more infrastructure costs. 

Further reading: 

[Rebuilding Netflix Video Processing Pipeline with Microservices | by Netflix Technology Blog](https://netflixtechblog.com/rebuilding-netflix-video-processing-pipeline-with-microservices-4e5e6310e359)

[Microservices.io](https://microservices.io/)

[What Is Microservices Architecture? | Google Cloud](https://cloud.google.com/learn/what-is-microservices-architecture)
