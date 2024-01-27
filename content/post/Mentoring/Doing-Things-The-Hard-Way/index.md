---
title: Doing Things The Hardway
description: Why you shouldn't always take the easy option when it's available.
date: 2024-01-27 00:00:00+0000
image: header-image.png
categories: [
    "Mentoring"
]
---

To be a well-rounded infrastructure engineer you need exposure and experience to various technologies. For example, networking, hardware, operating systems and applications. On the soft skills side, troubleshooting and problem solving.

To be candid, I think a lot of what I've done recently professionally has been quite easy - this sounds like a brag, it’s not meant to be. It’s an acknowledgement of how the public cloud makes consuming cloud services very easy relative to on-premises data centres.

As a passionate technologist (and an Azure cloud consultant professionally) I love the challenge of picking something I've not done before and trying to figure it out. That doesn't happen often for me anymore. But I know that’s not the case for everyone. Why is that? What's the difference between the start of my career and some of my colleagues?

I put it down to a lack of focus and training on the fundamentals of computing. For context I'm not talking about different types of gates within a processor like you might find on computer science courses at university. I'm sure that's very important for some professions but it’s not important for what I do professionally. I'm talking networking (OSI model, CIDR, subnets, routes, etc), Windows Server (Active Directory Domain Services, IIS, Hyper-V), and an understanding of public key infrastructure (PKI).

I can't, and don’t blame anybody for not knowing what I consider to be the fundamentals. On a day-to-day basis I don't need to know all of this. In Azure, if I need to get my virtual machine to go via a firewall before it goes to the Internet then I just tell Azure’s software defined networking via a user defined route to go to the frontend IP of an Azure firewall. Concepts like the default gateway are abstracted away. A positive argument to this abstraction is that it’s making cloud engineers more productive and enables more people to get involved with Azure, and I’m sure other cloud providers. But when things go wrong, how do people troubleshoot?

LLMs like ChatGPT, Bard, Llama are a fascinating technology! But I'm not sure that people are using them correctly when it comes to problem solving and troubleshooting. From what I've seen, whenever someone turns to an LLM to solve their problem, they just try to get an answer and not the understanding of why the solution works the way it does. They're likely doomed to face the same challenge when they're presented with the same problem but portrayed in a different way. If you're using LLMs to help you solve problems, make sure that you ask the LLM to explain the solution to you in way that you can understand it!

So, what am I doing about my lack of challenge? Well, I'm moving my blog from GitHub pages to a Kubernetes cluster hosted on Raspberry Pis. Why? There's a lot of technology that I need to understand that I have either not worked with for a long time or not at all!
By doing this I'll need to learn:

-	Physical infrastructure
-	Monitoring that infrastructure
-	Alerting when something goes wrong
-	Deploy and manage a self-hosted Kubernetes cluster
-	Host applications and services on Kubernetes
-	Monitor the applications (similar to infrastructure, I know)
-	Dockerise my blog, in a highly available way
-	Linux, a nemesis of mine

And I’m sure a lot more! I hope to blog about the challenges I face as I go through this process. See you on the other side! And I'd encourage you to think about where you can do things the hard way to become a more resilient person.
