---
title: Thoughts on web accessibility(信息无障碍)
date: 2020-01-31 21:00:52
categories: FrontEnd
tags:
    - Web Accessibility
    - FrontEnd
top:
---

我们在做前端的时候，实质上信息无障碍是很多工程师很容易忽视但又着实很重要的一部分，要知道在这个世界上是有很多人因为一些原因无法像正常人一样去浏览网页的，诸如色盲，盲人，老花等等，我们需要适配设计的网页，使其尽量对于各种人都是信息无障碍的。总觉得这是件大部分人都没有想到的事情，但是一旦想到了，那么就应该做点什么，来使这部分相对边缘的人也可以正常的去看我们设计的网页。

以下是Wiki给出的定义: 

> “ Web accessibility refers to the inclusive practice of removing barriers that prevent interaction with, or access to websites, by people with disabilities. When sites are correctly designed, developed and edited, all users have equal access to information and functionality.”


# 1. Accessibility 标准

+ 可感知
    + 如果只提供凭借一种感官才能让用户感知到内容，无形中会失去很多用户
+ 可操作
    + 能否正常使用每一个组件的功能
        + E.G 下拉菜单，很多网站设计的时候hover over的时候就有下拉效果，但是无法点击。 如果我们的用户无法看到这些东西，那很可能就无法继续交互下去了。
+ 可理解
    + 用户能否很好地理解呢？
        + 需要考虑读屏软件的适用性
+ 强健性
    + 能否被多种User Agent使用
        + 屏幕阅读器
        + IE


WebAIM (web accessibility in mind) [Checklist](https://webaim.org/standards/wcag/checklist) 

# 2. Tips 一些我们可以follow的东西

+ 标题 段落 列表 保持良好的结构
    + 屏幕阅读器在读到结构相对良好的标签的时候，会帮助用户更容易理解我们网站的内容
+ 尽可能使用语义化标签
    + 浏览器的调试工具当中有**Accessibility tree**,浏览器会获取DOM树，浏览器会获取DOM树，然后将其修改成适用于辅助技术的形式(无障碍树)，所以良好的使用语义化标签，能让辅助设备更合理地将我们网站的内容转化成Accessibility tree，从而解读给用户，确保页面中的重要的元素有正确的无障碍角色、状态和属性。
+ 为所有非文本内容提供文本替代项
    + 所有的图片都应当有alt属性，重要的图片应使用描述性替代文本简洁说明图像内容。
+ DOM顺序与Tab键顺序保持一致
+ 不要做`a {outline: none}`这种操作，因为这样的话这个小组件就是Unfocusable的了，对于不懂的人完全没法继续搞了
+ 对比度 最低要求 4.5:1 
+ 我们可以使用chrome浏览器的Audits找到自己的网站存在的所有无障碍问题，然后针对性的进行修改


# Reference
1. https://insights.thoughtworks.cn/about-web-accessibility/
2. https://webaim.org/standards/wcag/checklist 
