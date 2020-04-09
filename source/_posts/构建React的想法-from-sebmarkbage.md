---
title: 构建React的想法 - from sebmarkbage
date: 2020-04-08 20:25:08
categories: FrontEnd
tags:
    - React
top:
---
从github翻到的论述，是React设计者试图说明其构建React的整个逻辑。

# 1. 转换 - Transformation 

React的核心假定是UI层实际上是对数据的展现形式的一种转换。同样的输入会给出同样的输出结果，只是表现形式可能会有所区别。

    function NameBox(name) {
      return { fontWeight: 'bold', labelContent: name };
    }
    
    'Sebastian Markbåge' ->
    { fontWeight: 'bold', labelContent: 'Sebastian Markbåge' };

# 2. 抽象 - Abstraction 

我们无法将一个复杂的UI放到一个方法当中的。将UI抽象到各个可以复用的组件当中就变得尤为重要。抽象成可复用的组件，并且隐藏实现的细节，是我们想要做的。

    function FancyUserBox(user) {
      return {
        borderStyle: '1px solid blue',
        childContent: [
          'Name: ',
          NameBox(user.firstName + ' ' + user.lastName)
        ]
      };
    }
    
    { firstName: 'Sebastian', lastName: 'Markbåge' } ->
    {
      borderStyle: '1px solid blue',
      childContent: [
        'Name: ',
        { fontWeight: 'bold', labelContent: 'Sebastian Markbåge' }
      ]
    };

# 3. 组合 - Composition 

为了实现真正可以复用的特性，仅仅使用细分的子功能，并且每次给他们构建容器是不太够的。我们需要能够建立中间层的抽象，即将几个子功能组件组合起来，形成另一层次的组件。这里的组合就是指将多个抽象融合成一个抽象的能力。

    function FancyBox(children) {
      return {
        borderStyle: '1px solid blue',
        children: children
      };
    }
    
    function UserBox(user) {
      return FancyBox([
        'Name: ',
        NameBox(user.firstName + ' ' + user.lastName)
      ]);
    }

# 4. 状态 - State

UI不仅仅是服务器以及商业逻辑的复制，实际上是有很多状态，是服务于特定的运行过程的。比如你在一个文本框输入内容，那么我们需要一些方式能够记录下这个文本框当前的状态，并且用这个状态去和后端进或者其他的方法进行交互。

我们希望我们的数据是immutable的

    function FancyNameBox(user, likes, onClick) {
      return FancyBox([
        'Name: ', NameBox(user.firstName + ' ' + user.lastName),
        'Likes: ', LikeBox(likes),
        LikeButton(onClick)
      ]);
    }
    
    // Implementation Details
    
    var likes = 0;
    function addOneMoreLike() {
      likes++;
      rerender();
    }
    
    // Init
    
    FancyNameBox(
      { firstName: 'Sebastian', lastName: 'Markbåge' },
      likes,
      addOneMoreLike
    );
    
# 5. 记忆方法
如果我们知道这是个纯方法，并且还会重复的call它，那么我们可以设计一个记忆版本的方法，这样我们就不用在有相同的数据(输入)的时候还要重复执行了

    function memoize(fn) {
      var cachedArg;
      var cachedResult;
      return function(arg) {
        if (cachedArg === arg) {
          return cachedResult;
        }
        cachedArg = arg;
        cachedResult = fn(arg);
        return cachedResult;
      };
    }
    
    var MemoizedNameBox = memoize(NameBox);
    
    function NameAndAgeBox(user, currentTime) {
      return FancyBox([
        'Name: ',
        MemoizedNameBox(user.firstName + ' ' + user.lastName),
        'Age in milliseconds: ',
        currentTime - user.dateOfBirth
      ]);
    }

    // 我们同样可以不仅仅记一个值，也可以记一个map
    function memoize(fn) {
      return function(arg, memoizationCache) {
        if (memoizationCache.arg === arg) {
          return memoizationCache.result;
        }
        const result = fn(arg);
        memoizationCache.arg = arg;
        memoizationCache.result = result;
        return result;
      };
    }
    
    function FancyBoxWithState(
      children,
      stateMap,
      updateState,
      memoizationCache
    ) {
      return FancyBox(
        children.map(child => child.continuation(
          stateMap.get(child.key),
          updateState,
          memoizationCache.get(child.key)
        ))
      );
    }
    
    const MemoizedFancyNameBox = memoize(FancyNameBox);
    
# 6. 列表

大部分的UI都是一些形式的列表，然后为在列表中的每个元素产出不同的一系列的值。这就自然的产生了一个有层级的结构。

我们可以通过使用Map方法来管理每个列表当中的元素的状态。

    function UserList(users, likesPerUser, updateUserLikes) {
      return users.map(user => FancyNameBox(
        user,
        likesPerUser.get(user.id),
        () => updateUserLikes(user.id, likesPerUser.get(user.id) + 1)
      ));
    }
    
    var likesPerUser = new Map();
    function updateUserLikes(id, likeCount) {
      likesPerUser.set(id, likeCount);
      rerender();
    }
    
    UserList(data.users, likesPerUser, updateUserLikes);
    
# 7. 持续性

一些时候，我们在运行我们的核心商业逻辑的时候会大量的操作数据，这部分关于数据的操作会显得有些冗余，我们可以将其移出核心逻辑的代码块，比如使用bind，来绑定方法，在其他地方写具体的代码逻辑。

    function FancyUserList(users) {
      return FancyBox(
        UserList.bind(null, users)
      );
    }
    
    const box = FancyUserList(data.users);
    const resolvedChildren = box.children(likesPerUser, updateUserLikes);
    const resolvedBox = {
      ...box,
      children: resolvedChildren
    };
    
# 8. 状态Map

我们可以用组合来将几个子组件放在一起来使用，同样的，对于他们需要的一些输入数据，我们可以通过state，来传到下层的方法处，供他们使用。

    function FancyBoxWithState(
      children,
      stateMap,
      updateState
    ) {
      return FancyBox(
        children.map(child => child.continuation(
          stateMap.get(child.key),
          updateState
        ))
      );
    }
    
    function UserList(users) {
      return users.map(user => {
        continuation: FancyNameBox.bind(null, user),
        key: user.id
      });
    }
    
    function FancyUserList(users) {
      return FancyBoxWithState.bind(null,
        UserList(users)
      );
    }
    
    const continuation = FancyUserList(data.users);
    continuation(likesPerUser, updateUserLikes);

# Reference
https://github.com/reactjs/react-basic
