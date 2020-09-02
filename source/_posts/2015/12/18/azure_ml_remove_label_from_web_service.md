---
title: Azure ML, remove label from web service
date: 2015-12-18
tags: 
- Azure
- Azure Machine Learning
---

Let's take the following simple experiment: 

![](/images/151218b/1.png)

that you publish as a web service

![](/images/151218b/2.png)

When you call the web service, you are required to provide the answer to the question you are asking (even if Azure ML won't use it): 

![](/images/151218b/3.png)

the solution is quite simple: project columns to remove the label from the input: 

![](/images/151218b/4.png)

and you'll get a better question

![](/images/151218b/5.png)

:-)
Benjamin (@benjguin)