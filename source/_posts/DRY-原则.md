---
title: DRY 原则
date: 2020-03-21 20:01:51
categories: SystemDesign
tags:
top:
---
Don't repeat yourself 

# 1. 实现逻辑的重复


    public class UserAuthenticator {
      public void authenticate(String username, String password) {
        if (!isValidUsername(username)) {
          // ...throw InvalidUsernameException...
        }
        if (!isValidPassword(password)) {
          // ...throw InvalidPasswordException...
        }
        //...省略其他代码...
      }
    
      private boolean isValidUsername(String username) {
        // check not null, not empty
        if (StringUtils.isBlank(username)) {
          return false;
        }
        // check length: 4~64
        int length = username.length();
        if (length < 4 || length > 64) {
          return false;
        }
        // contains only lowcase characters
        if (!StringUtils.isAllLowerCase(username)) {
          return false;
        }
        // contains only a~z,0~9,dot
        for (int i = 0; i < length; ++i) {
          char c = username.charAt(i);
          if (!(c >= 'a' && c <= 'z') || (c >= '0' && c <= '9') || c == '.') {
            return false;
          }
        }
        return true;
      }
    
      private boolean isValidPassword(String password) {
        // check not null, not empty
        if (StringUtils.isBlank(password)) {
          return false;
        }
        // check length: 4~64
        int length = password.length();
        if (length < 4 || length > 64) {
          return false;
        }
        // contains only lowcase characters
        if (!StringUtils.isAllLowerCase(password)) {
          return false;
        }
        // contains only a~z,0~9,dot
        for (int i = 0; i < length; ++i) {
          char c = password.charAt(i);
          if (!(c >= 'a' && c <= 'z') || (c >= '0' && c <= '9') || c == '.') {
            return false;
          }
        }
        return true;
      }
    }


这里想强调的是不一定完全一样的代码就意味着他们是需要合并的，比如上述的代码当中，isValidPassword还有isValidUsername有着基本上相同的逻辑结构，但是他们本身代表的是不同的意思的，虽然一样的逻辑，但是我们无法保证在接下来的一段时间以内，他们还能这样子一致下去。

所以合并为isValidUserOrPassword是不可以的，逻辑上讲不通，但是我们可以将上面在这个函数内部调用的方法进行分割的方式，来复用一些代码。

其实有一些使用组合的味道在里面了。


# 2. 功能语义的重复

对于功能语义重复的理解，是指一个代码包里在多个地方为了实现同样的功能，设定了不同的方法体。这大部分情况下都是因为沟通等方面的问题，不管里面使用的方法有什么不同，如果他们都是为了达成一样的，那么我们认为其实违反了DRY原则的，是需要清除多余的方法体的。

# 3. 代码执行重复

看代码的内部逻辑，减少多次重复调用的代码。

注意IO操作，因为IO非常费时...
