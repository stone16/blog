---
title: Git 流程
date: 2020-08-11 17:41:26
categories: BackEnd
tags: 
    - git 
top:
---
解决git conflict永远都是件很让人头疼的事情，为了让生活更简单，还是需要设定正确的git流程的。现在有如下几种git 流程

# 1. 基本的Git 流程

只有一个branch -- master. 开发者直接commit进去，然后会进入到alpha，beta, gamma, prod等不同的生产状态当中。

一般来说，除非你在自己单独完成某项小任务，是很不推荐这样做的。

缺陷在于：
+ 代码上的合作变得很困难，可能会有多次冲突，需要逐次进行解决


# 2. Git feature分支流程
当在同一个codebase我们有多个工程师共同工作的时候，使用feature分治就变成了必不可少的事情了。

如果现在有两个工程师在同一个branch上工作，来提交自己的代码，那最终一定是冲突不断的，很容易出现各种问题。

为了避免出现这种情况，两个开发者可以创建两个不同的分支，分别在自己的分治上来开发自己的项目。

这样做的好处是不用担心大量需要解决的冲突了。

# 3. Git feature分支流程与Develop分支

和上述的feature分支流程很类似，只是又加了一个Develop分支，在这个流程下，master 分支永远反映一个prod ready的状态。

无论何时，当小组想要将代码部署到prod的时候，他们从master分支来进行部署

develop branch反映的是带着最新的为了下次发布准备的所有改动。开发者fork develop 分支的代码，来做独立开发。一旦项目做好，经过了测试，就合并到develop分支当中，在develop分支来做充分的测试，然后再merge到master分支当中去。


这样做的好处是能够允许小组持续merge新的功能，做持续集成。不过过程相对比较麻烦。个人观点是在小规模的前提下，使用特征分支就足够了，再加上持续集成的工具，譬如Jerkins，很安全，效率也很不错。

https://zepel.io/blog/5-git-workflows-to-improve-development/