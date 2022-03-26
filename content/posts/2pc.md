---
title: "Two phase commit ?"
date: 2022-03-26 01:36:57.216992
draft: false
cover: https://file.coffee/u/Wgn6g2ZF9OgpdS.png
---


## Two phase commit? 
 Lately I have found myself pondering how distributed systems handle transactions, particularly what happens if one database replica is out of sync, how should the "ideal" distributed system handle such?

Turns out there are few ways of handling distributed transactions, and two phase commit or 2PC is one of them. So I am doing the logical thing and writing down what I have managed to learn at 2:32am on a weekday.

## 2PC you say  ?! 
![wutt](https://file.coffee/u/U4Ik7C5uGiHTBK.jpg)

First I think it's important to discuss what problem 2PC is trying to address before diving into how it solves it. 

## The problem 
When updating data across multiple nodes a prominent problem is nodes going out of sync or becoming inconsistent , so how do we ensure all nodes receive the same copy of data when update is performed?

## Solution 
As the name suggests, two phase commit performs an update in two phases:

- Prepare phase
- Commit phase 

#### Prepare phase 
In this phase all participating nodes are asked if they are able to carry out the commit by the transaction coordinator, the transaction co-ordinator is responsible for ensuring all nodes are able to perform the commit , if one node is unable is unable to carry out the commit the transaction is rolled back and marked as unsuccessful.

#### Commit phase
The second phase begins once all participating nodes are able to perform the commit. Once the commit is performed each node sends a message to the transaction co-ordinator letting it know that operation has been completed, once all participating nodes respond with a success message the transaction is marked as complete and the commit phase ends. 



### Closing thoughts 
Obviously this was a high level overview of what 2pc is and I do not go into specifics of how 2pc works, but I think thats ok. At the beginning of this post I had no clue what two phase commit was, I'd say this is a decent explanation for anyone who isn't concerned with the fine details and just wants to answer the burning question WTF is 2pc. 

But if you'd like to know more here are some relevant links I found while researching this: 

From Martin Fowler
https://martinfowler.com/articles/patterns-of-distributed-systems/two-phase-commit.html#solution

Obligatory wikipedia reference
https://en.wikipedia.org/wiki/Two-phase_commit_protocol

This blog I found 
https://www.techopedia.com/definition/1252/two-phase-commit

Not as comprehensive but helpful 
https://www.educative.io/edpresso/what-is-the-two-phase-commit-protocol









