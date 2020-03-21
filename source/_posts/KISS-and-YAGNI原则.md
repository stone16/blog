---
title: KISS and YAGNI原则
date: 2020-03-20 20:24:37
categories: SystemDesign
tags:
top:
---
# 1. KISS
Keep it simple and stupid. 

这个原则相对比较范范，其实着重在说的还是从代码的可读性和可维护性两个角度来衡量代码的质量。


    // 第一种实现方式: 使用正则表达式
    public boolean isValidIpAddressV1(String ipAddress) {
      if (StringUtils.isBlank(ipAddress)) return false;
      String regex = "^(1\\d{2}|2[0-4]\\d|25[0-5]|[1-9]\\d|[1-9])\\."
              + "(1\\d{2}|2[0-4]\\d|25[0-5]|[1-9]\\d|\\d)\\."
              + "(1\\d{2}|2[0-4]\\d|25[0-5]|[1-9]\\d|\\d)\\."
              + "(1\\d{2}|2[0-4]\\d|25[0-5]|[1-9]\\d|\\d)$";
      return ipAddress.matches(regex);
    }
    
    // 第二种实现方式: 使用现成的工具类
    public boolean isValidIpAddressV2(String ipAddress) {
      if (StringUtils.isBlank(ipAddress)) return false;
      String[] ipUnits = StringUtils.split(ipAddress, '.');
      if (ipUnits.length != 4) {
        return false;
      }
      for (int i = 0; i < 4; ++i) {
        int ipUnitIntValue;
        try {
          ipUnitIntValue = Integer.parseInt(ipUnits[i]);
        } catch (NumberFormatException e) {
          return false;
        }
        if (ipUnitIntValue < 0 || ipUnitIntValue > 255) {
          return false;
        }
        if (i == 0 && ipUnitIntValue == 0) {
          return false;
        }
      }
      return true;
    }
    
    // 第三种实现方式: 不使用任何工具类
    public boolean isValidIpAddressV3(String ipAddress) {
      char[] ipChars = ipAddress.toCharArray();
      int length = ipChars.length;
      int ipUnitIntValue = -1;
      boolean isFirstUnit = true;
      int unitsCount = 0;
      for (int i = 0; i < length; ++i) {
        char c = ipChars[i];
        if (c == '.') {
          if (ipUnitIntValue < 0 || ipUnitIntValue > 255) return false;
          if (isFirstUnit && ipUnitIntValue == 0) return false;
          if (isFirstUnit) isFirstUnit = false;
          ipUnitIntValue = -1;
          unitsCount++;
          continue;
        }
        if (c < '0' || c > '9') {
          return false;
        }
        if (ipUnitIntValue == -1) ipUnitIntValue = 0;
        ipUnitIntValue = ipUnitIntValue * 10 + (c - '0');
      }
      if (ipUnitIntValue < 0 || ipUnitIntValue > 255) return false;
      if (unitsCount != 3) return false;
      return true;
    }

相对来说，第二种实现方式会更好一些，因为细节被封装在工具类当中，可读性相对强很多；方法一直接使用正则表达式，可读性会差很多；第三种自己来处理底层的逻辑，虽然执行起来相对会快一些，但是很容易出错。

# 2. YAGNI
You ain't gonna need it. 

你不会需要它。深有感触，很多时候我们想写很优雅的代码，会疯狂向后考虑，比如对于不常用的功能，有的时候哪怕就一个方法，也想搞个接口供后面的扩展来使用。过度设计，反而增加了代码的阅读成本和维护成本。

我们不应当去设计当前用不到的功能，也不应该编写现在用不到的代码。