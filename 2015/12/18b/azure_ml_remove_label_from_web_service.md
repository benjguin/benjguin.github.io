#Azure ML: remove label from web service

Let's take the following simple experiment: 

![](img/1.png)

that you publish as a web service

![](img/2.png)

When you call the web service, you are required to provide the answer to the question you are asking (even if Azure ML won't use it): 

![](img/3.png)

the solution is quite simple: project columns to remove the label from the input: 

![](img/4.png)

and you'll get a better question

![](img/5.png)

:-)
Benjamin (@benjguin)