---
title: 2'nd Year Working in Cyber Security
date: 2024-10-05 8:16:00 -5000
categories: [Cyber Security]
tags: [cyber security]     # TAG names should always be lowercase
---


# Description

It's now been 2 years of working in cyber security, and I’m writing this blog to reflect on what I’ve learned during my second year. If you haven’t already, check out my [1st-Year-Working-in-Cyber-Security](https://joshmorrison99.github.io/posts/1st-Year-Working-in-Cyber-Security/) to get some background on where I started and how things have evolved.

This past year has brought significantly more responsibility and a heavier workload compared to my [1st-Year-Working-in-Cyber-Security](https://joshmorrison99.github.io/posts/1st-Year-Working-in-Cyber-Security/). My primary focus was developing a vulnerability database for open-source software. This involved collecting and parsing data from a wide range of sources, including REST APIs, GraphQL endpoints, SOAP APIs, OVAL feeds, CSAF feeds, RSS feeds, Git repositories, and through web scraping. Once collected, the data went through extensive comparison and validation—mostly using Power BI and Python. 

In addition to that, I contributed to enhancing and fixing bugs in existing DevOps pipelines, and worked on migrating an on-premise project to the cloud. I also took on a few side projects, including AI-based secret detection using LLaMA 3, building a knowledge base visualization website, and experimenting with DLL injection in a competitor’s product.

# Major Learnings

| Problem | Solution | Learnings |
| --- | --- | --- |
| Needed to convert a 4GB JSON file into a SQLite3 database quickly and without excessive RAM usage. | Use an iterative JSON parsing library - https://pypi.org/project/ijson/ | - When python opens a file, the entire 4GB file is read into memory. <br/> - Using SQLite3 and learning how powerful it can be. <br/> - I had also wanted to learn Rust so I re-wrote the python converter in rust which had a great speed and RAM usage improvement. |
| Parsing OVAL Feeds | Reading lots of documentation - https://oval.mitre.org/ | - The Open Vulnerability and Assessment Language (OVAL) is not a very straight forward schema. |
| Comparing our Data to Competitors | Created a Power BI Pipeline and Report to get daily comparisons on how our product compares to our competitors. | - How to create reports and pipelines using PowerBI. |
| The 4GB JSON file cannot be opened in Browser or VSCode due to file size. This makes it hard for us and other teams at check our data. | Created a portal using React.js, AWS API Gateway, RDS and Lambda to allow user to query our JSON data. | - How to use API gateway, restricting to only certain IPs. <br/> - How to use Lambda to query RDS and and interact with API gateway. |
| Uploading to dev, stg, and prod become misaligned due to people not updating repository. | Create a GitHub action that would automatically build, validate and upload docker images to corresponding ECR environment based on what branch the PR is being merged into. | - Learned how to use more complicated GitHub actions. <br/> - Learned how important it is to have one source of truth for a project and not letting anyone push to ECR with hopes that the GitHub repository will also be updated. |
| Debugging the Microsoft KB sequence is difficult and time consuming. | Created a website that will visualize the KB sequences of different products allowing us to debug and share findings easier. | - Learned more about the complexities of KB sequences. |
| The older versions of our product no longer supports the newest kernel versions that our pipelines use. These are large pipelines that cannot be re-written. | Rather than building on the latest kernel, I created an AMI from older linux distributions that contained the supported kernel. | - Learned how to create an AMI.  |

Some of the less notable learnings are:

- Learned more on how to use JFrog.
- Learned more on Jenkins (Using different Jenkins plugins and discovering new Jenkins features)
- Fine-tuned Llama3 to detect secrets in code repositories. Project never took off, it was only investigation but it was fun learning about AI.
- Helped in replicating a few undisclosed vulnerabilities to check if our product was able to detect it.
- Learned how to use GitHub action to validate JSON using maven and AJV (npm package).
- Learned about how to interact with AWS S3 data across AWS accounts.
- Learned about using AWS EventBridge to notify other teams.
- Learned how to use Microsoft Workflows to send Teams messages when O365 teams notifications was deprecated.
- Learned more about CSAF feeds, SOAP APIs, web-scraping and GraphQL.

> There is a lot more things I had done this year, but I’m only outlining the learnings.


![PowerBI](/assets/powerbi.png)
*Comparing our dataset with a competitors in PowerBI*

![vulnerability-portal](/assets/vulnerability-portal-0.png)
*Vulnerability portal - search results*

![vulnerability-portal](/assets/vulnerability-portal-1.png)
*Vulnerability portal - more information*

![vulnerability-portal](/assets/kb-visualization.png)
*Microsoft KB visualization website*
