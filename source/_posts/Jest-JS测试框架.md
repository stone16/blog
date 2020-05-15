---
title: Jest - JS测试框架
date: 2020-05-13 21:31:23
categories: FrontEnd
tags:
    - Jest
top:
---
JJest是一个简洁的JavaScript测试框架，我们可以将其与Babel, TS, Node, React, Angular, Vue等来共同使用

# 1. Intro

首先jest是希望能够用很轻量的方式来进行前端测试，自己最近在做一个偏向前端的项目，希望把组里的对外页面从其他平台转移出来，通过router完成路径的转接，然后在自己的平台上，就可以更自由，更快捷的进行迭代了。

整个架构还是RPC 暴露RESTFul接口，用类似于API Gateway的系统完成Authorization Authentication的工作，前端直接调用后端的信息这样子。想要达到的最终的目标就是易于维护且易于扩展的一个前端小平台，几个基本需求，也是要做转移的原因：

+ 前端的埋点，希望有更多的metrics以知道用户的行为
    + 各个页面的浏览时长
    + 跳出率
    + 哪个步骤过滤走了最多的用户请求，诸如此类

+ CI/ CD
    + 手动QA太容易犯错了，如果能写一部分unit tests & integration tests,实现整个前端页面的可测试，那么就可以实现持续继承持续部署，会很大程度上提高可交付能力

+ 访问速度
    + 利用S3 host页面，使用CDN完成分布，会提高整体的响应速度


Jest是实现CI/CD的很不错的一个工具，on my way learning it ;) 

## 1.1 匹配器

与Junit类似，使用expect做关键词，E.G

    test('two plus two is four', () => {
      expect(2 + 2).toBe(4);
    });

+ toBe 
    + 匹配器，内部使用的是`Object.is`来做精确相等
+ toEqual
    + 来检查对象的值

+ 比较真实性
    + toBeNull
    + toBeUndefined
    + toBeTruthy
        + 匹配任何if语句为真
    + toBeFalsy 
        + 匹配任何if语句为假 

+ 比较数字
    + toBeGreaterThan()
    + toBeGreaterThanOrEqual()
    + toBeLessThan()
    + toBeLessThanOrEqual()
    + toBe()
    + toEqual()

+ 字符串
    + toMatch()  match一个正则表达式

+ 数组 
    + toContain() 检查一个数组或可迭代的对象是否包含某个特定项


    const shoppingList = [
      'diapers',
      'kleenex',
      'trash bags',
      'paper towels',
      'beer',
    ];

    test('the shopping list has beer on it', () => {
      expect(shoppingList).toContain('beer');
      expect(new Set(shoppingList)).toContain('beer');
    });
## 1.2 测试异步代码

    // 对于回调函数的测试
    test('the data is peanut butter', done => {
      function callback(data) {
        try {
          expect(data).toBe('peanut butter');
          done();
        } catch (error) {
          done(error);
        }
      }

      fetchData(callback);
    });

    // Promises
    test('the data is peanut butter', () => {
      return fetchData().then(data => {
        expect(data).toBe('peanut butter');
      });
    });

    // Resolve/ reject
    test('the data is peanut butter', () => {
      return expect(fetchData()).resolves.toBe('peanut butter');
    });
    
    // async/ await
    test('the data is peanut butter', async () => {
      const data = await fetchData();
      expect(data).toBe('peanut butter');
    });

    test('the fetch fails with an error', async () => {
      expect.assertions(1);
      try {
        await fetchData();
      } catch (e) {
        expect(e).toMatch('error');
      }
    });
    
## 1.3 测试前后的utility方法

+ 重复设置值 


    beforeEach(() => {

    });

    afterEach(() => {

    });
    
+ 一次性设置 -- 单个测试不会改变其值

    beforeAll(() => {

    });

    afterAll(() => {

    });

+ 作用域 
    + 通过describe来将测试进行分组操作 
    + 注意describe的执行顺序 
        + 在真正的测试开始之前执行测试文件当中的所有的describe处理程序
        + 当describe块运行完后，Jest会按照test出现的顺序依次运行所有测试，等待每一个测试完成并整理好，然后继续往下走
    + 通用建议
        + 当测试失败的时候，首先要检查的是如果仅运行这条测试，是否仍然失败
        + 通过将test指令改为test.only指令来实现


## 1.4 Mock 方法

Mock函数允许我们来测试代码之间的连接，和Mockito， EasyMock其实是一个理念的，擦除函数的实际实现，专注于当前的文件的方法本身，捕获对函数的调用，实例等

    function forEach(items, callback) {
      for (let index = 0; index < items.length; index++) {
        callback(items[index]);
      }
    }

    const mockCallback = jest.fn(x => 42 + x);
    forEach([0, 1], mockCallback);

    // 此 mock 函数被调用了两次
    expect(mockCallback.mock.calls.length).toBe(2);

    // 第一次调用函数时的第一个参数是 0
    expect(mockCallback.mock.calls[0][0]).toBe(0);

    // 第二次调用函数时的第一个参数是 1
    expect(mockCallback.mock.calls[1][0]).toBe(1);

    // 第一次函数调用的返回值是 42
    expect(mockCallback.mock.results[0].value).toBe(42);

# 2. 测试方法
## 2.1 Snapshot 测试

给当前的UI做快照，然后和过去做过的快照进行比较，看是否有不同。

    import React from 'react';
    import Link from '../Link.react';
    import renderer from 'react-test-renderer';

    it('renders correctly', () => {
      const tree = renderer
        .create(<Link page="http://www.facebook.com">Facebook</Link>)
        .toJSON();
      expect(tree).toMatchSnapshot();
    });

实际上是生成一个DOM树，然后来比较两颗DOM树的节点，看设置是否相同。

在每次提交的时候，会记录下当前的快照，下次提交的时候会和这次的来进行比较。

然后当我们有目的的引入了变化的时候，我们需要告诉jest 需要更新现在保存的snapshot了，这种情况下需要运行指令`jest --updateSnapshot`.


# Reference 
1. https://jestjs.io/docs/en/getting-started 