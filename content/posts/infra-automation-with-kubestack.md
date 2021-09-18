---
title: "Infrastructure Automation With Kubestack"
date: 2021-06-07T19:15:43+01:00
draft: false
cover: https://repository-images.githubusercontent.com/161867918/8023b100-c1a8-11e9-8296-b63c3c068427
---


# Infrastructure Automation with Kubestack

Over the weekend I got to try out [Kubestack](https://kubestack.com) and  in this post, I would be giving some of my thoughts on it.

FULL DISCLAIMER: while I am being paid to write this post the opinions  and views expressed here are completely mines.

## So what's this thing anyway?

That was my first question as I have only ever heard about Kubestack in  passing.

Kubestack is a [GitOps](https://about.gitlab.com/topics/gitops/)  framework aimed at simplifying the process of spinning up a Kubernetes  cluster and automating subsequent deployments to your cluster. At the  time of writing, Kubestack supports GKE, AKS, and EKS. Alright, now I  know what it is but how is this any different than building my own CI/CD  pipeline? Only one way to find out...

## Getting started

Before this, I have only made one attempt at automating how I deploy  Kubernetes on azure, so if this worked out it would be such a joy  because my last experiment didn't go too well. So for the majority of  this process, I would keep comparing my experience to my last attempt at  something like this.

Enough talk let's get right to it. The quickest way of getting started  with Kubestack is by going through the [getting  started](https://www.kubestack.com/framework/documentation/tutorial-get-started)  guide. The guide is split into three parts which I find very nice as it  allowed me to digest an individual section before moving to the next.

### Kubestack command line

The setup here was quite straightforward, grab the binary, unzip and  move it to your path, before I proceeded with the rest of the tutorial I  decided to explore a bit by viewing some of the subcommand available.  Honestly, I have no real complaints here, the subcommands are clearly  explained and flags are consistent. Although I would like to see tab  completion, that's just personal preference 😄

### Bootstrapping your cluster

Creating a cluster with Kubestack gets even easier as it automatically  generates the base terraform

code. This is powerful because one you can now generate the  configuration for all supported clouds and two I no longer have to copy  and paste most of my code from a previous project. Once you have your  "repository" initialized you only have to change three values to get  going.

Where things started to get tricky for me was the inheritance model the  terraform code Kubestack generated uses, But this might be due to my  lack of experience in that area but it's something to look out for.

### Testing locally

Kubestack has a neat feature that will provision a local Kubernetes  cluster using [KinD](https://kind.sigs.k8s.io/) and test your  configuration against it, the advantage of this can't be emphasized  enough, I've broken a good chunk of my infrastructure simply because I  didn't have a sandbox environment to test against.

In addition, Kubestack generates a docker image with a few tools  installed e.g(Kustomize, Terraform)

I found this quite convenient as I had an older version of terraform  installed so I just opted to use the docker image instead. Creating a  service principal for my account was a fairly smooth experience, the  only real problem I ran into was giving to my service account. I ran  into the following  [error](https://login.microsoftonline.com/error?code=50076) which I was  able to resolve by granting the account permission through the Azure  portal.

## Deploying

This was equally as easy creating two workspaces and deploying resources  in each was pretty straightforward, one new concept (for me at least) is  Kubestack would set up DNS per environment so this interesting to see.

## Automation

This was the part I was looking forward to the most. The goal is simple.  Have a GitHub action setup in such a way that I can deploy to different  environments and the pipeline would apply the Configuration.

#### Did it work?

Yes! Of all the steps this took me the least amount of time to complete,  the only problem I ran into was base64 encoding my credentials which I  got around by removing `-w` because line breaks are automatically ignored on Mac, even at that it took me about 10mins to set up the  entire pipeline.

While I'm a huge fan of Github actions it would be nice to have the  option of seeing the setup for other CI services like GitLab

## Beyond the tutorial

And now for the fun part, trying things outside the tutorial. For this  part, I wanted to see what else I could deploy to my cluster so I  consulted the [catalog](https://www.kubestack.com/catalog) section and  picked Argo CD ( other resources such as Flux Prometheus Operator and Cert Manger are also available ), as I mentioned earlier I'm very new to the inheritance  model Kubestack uses and so this did not end too well, but this was not  a total failure I saw this as an opportunity to really stretch the  Pipeline I had just built out and so I made one more failing deployment  and then I was able to rollback by simply reverting my commit, this is  where GitOps truly shines, the fact that your infrastructure can now be  thought of as commits in Git, Although I think the documentation could  do a better job towards guiding people new to this style of  configuration.

## Additional concerns

As I worked my way through the tutorial one thing that bothered me were the deprecation warnings that popped up every now and then, as i mentioned earlier this is something that could be fixed in a subsequent release.

## Closing thoughts

At the end of the entire process I had but one question.

Was it worth it? Yes, it was. this is hands down the quickest I've ever  set up something like this the entire process took me about an hour,  compared to my previous attempt which took me three days :). So this is a huge saving in effort and time, a few additional things i'd like to see are: 

- more resources in the catalouge section. 
- more clouds supported. 


That sums up my experience with  Kubestack feel free to check it out over [here](https://kubestack.com)  and the complete code from the tutorial over  [here](https://github.com/s1ntaxe770r/kubestack-sandbox)

