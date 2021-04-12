# Why you might want to have an end to end walkthrough document in your repo

As we are wrapping up an engagement where we've been coding with engineers and data scientists in a TelCo company,
we are sharing some of the learnings from this engagement.

That's part of our team's mission: We code with high impact organization developers to bootstrap the projects they’ll bring to production in Azure and pave the way for others.

You'll learn more from the same engagement at <https://aka.ms/hat-sharing>

## Context

In the engagement we were a larger team than usual. Customer had 7 developers and data scientists, 1 subject matter expert. MS brought 1 Business PM, 1 Technical PM, 1 Dev lead, 1 Data Science Lead, 2 Tech Leads (I was one of the two), 7 developers, and 2 data scientists.

That's definitely more than a pizza team! (Well, we were working from home, so each team member could order a pizza for themselves and nobody starved anyway...)

Because the team was large we split the team based on the epics we had in the backlog. A small team of ~3 would take an epic and take ownership from design to implementation. An epic would be about data transformation pipeline, another one about data science pipeline, another one about clustering, etc. There were also cross epic design meetings.

An additional piece of context is that [Microsoft CSE (Commercial Software Engineering)](https://github.com/microsoft/code-with-engineering-playbook/blob/master/CSE.md) codes with customers. The customer team is the catcher, they move on after we leave. If we didnt come to that point when we leave, they are the ones bringing the project to production. So it's important the code we leave is cristal clear for the catcher team.

## Impediments

We had two important obstacles while running the engagement:

1. Data needs evolved and the datasets we had became partially obsolete. We didn't get the right new datasets early enough. Because of that, parts of the project used small datasets (1 day) with the new schema, while other parts of the project stayed with medium datasets (few weeks) that allowed clustering to work.
1. We didn't get access to the target pre-production environment early enough. Because of that, we recreated a similar environment in our development Azure subscription, besides the development environment. Let's call it the simulated target environment.

Without us figuring out, the team started to progress in parallel streams that may not converge. There were unit tests, and plans to develop some integration tests. There was no clear target we could touch, no clear artifact you test, fix and enhance. There was only that mental picture each team member had on top of their mind; was the picture the same for each of us? Was the team still walking in the right and same direction?

## Integration tests early in the game

It was time to put higher priority on and end to end integration test running in an Azure DevOps pipeline that everyone could see, touch, fix and enhance. It was also important to have this end to end integration test to make it more obvious when we had to switch datasets. For instance, data transformation pipeline used new raw datasets (with the new schema) to go to bronze, silver and gold datasets, but the gold dataset could not be used for clustering because there was only one day of data while a few weeks were needed. The data science pipeline used an old datasets that had enough data. We still included that in the end to end integration test because the main goal was to reflect what we had built so far, and have something to fix, not be content with different parts that individually worked well.

If i had to run the same engagement again, I would have an integration tests Azure DevOps pipeline much earlier.

## End to end walkthrough document

The end to end integration test pipeline didn't reflect the work from the whole team. There were people working on the development environment itself, others working on data science enhancements (what would be the best clustering algorithm, how would you interpret the clusters, how to detect concept drifts), others on data visualization. All this was not part of the integration tests pipeline.

We already had a lot of documentation in the repo (e.g. design documents written by the epics teams) but that still didn't provide a quite concise yet holistic view of what was in the repo.

Having an end to end walkthrough document is a way to have a sequential list of things to run, test and read to get up to speed on what the repo contains. Entering the code of a repo is hard. What if we could have a tutorial that walks a new team member through the code and explains how to run it?

For the developers, as they write paragraphs in that document, it's also a way to make it even more obvious how their work fits in the overall picture.

The end to end walkthrough document has a broader scope than the integration tests. Its central piece is about running the integration tests pipeline and documenting what it does. Offloading most of the code to Azure DevOps pipelines makes it easier to maintain the document. That's quite obvious for the integration tests but that also works for things like dataset exploration; instead of relying on the reader to use their development environment and run code to explore datasets, why not have that code in an Azure DevOps pipeline? That makes it much easier to check that the code is still up to date (just run the pipeline).

So, what does such a document look like?

Here is the table of content of the end to end walkthrough document for the hat engagement:

- Development environment (how to get sources, setup and run your dev container, check that everything works correctly for you)
- Explore datasets [1]
- Unit Tests (how to run the unit tests from the dev environment)
- End to end integration tests: data transformation pipeline, data science pipeline, concept drift detection pipeline, recommendation pipeline (how to run the Azure DevOps integration test pipeline and read the results)
- Execute Data Engineering and Data Science Pipelines (how to run CD Azure DevOps pipeline and run the Databricks jobs in the target environment)
- DataViz (how to see results in Power BI)
- Continuous Integration - CI (understand what the CI pipeline does)
- Continuous Deployment - CD (understand what the CD pipeline does)
- Data Science Experiments (how to run the data science notebooks) [2]

[1] an Azure DevOps pipeline is associated to that part of the document. It allows to get datasets from the dev environment's data lake and extract schema and a few rows to get a glance on how the dataset looks like without running an actual notebook. This cannot be seen directly from Azure Data Explorer as datasets would come in formats like ORC, CSV in tar.gz or parquet.

[2] Data scientists worked in notebooks (experiment notebooks), then code was translated by developers into unit tested Python code + bootstrap notebooks. The experiment notebooks were stored in git in a separate folder.

## Conclusion

If I had to go back in time, I would have integration tests in an Azure DevOps pipeline much sooner and start the end to end walkthrough document very early also. Any important pull request that would bring new features would update both.

I would also use parts of the end to end walktrhough document to have descriptions of what the code would do in the near future. That's complementary to the backlog which is at a lower level of details.

**Both the end to end walkthrough document and integration tests in an automated pipeline are a great target the whole development team can share, look at, touch, run, fix, enhance.**
