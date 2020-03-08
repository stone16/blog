---
title: 基于充血模型的DDD开发模型
date: 2020-03-05 16:49:16
categories: SystemDesign
tags:
    - DDD
top:
---
# 1. 传统基于MVC的开发模式

MVC三层结构中的M表示Model，V表示View，C表示Controller。通过这三层将整个项目分为了三大部分，展示层，逻辑层，数据层

而贫血模型 - Anemic Domain Model，指的是我们将数据和操作分离，有专门的POJO类，即只包含数据的类，这实质上会破坏面向对象的封装特性，是一种面向过程的编程风格。

# 2. 基于充血模型的DDD开发模式

充血模型 - rich domain model，旨在将数据和对应的业务逻辑封装在同一个类当中。

# 3. 实战  DDD 开发虚拟钱包系统 

## 3.1 钱包业务背景介绍

需要创建一个系统内的虚拟钱包账户，来支持用户的充值，提现，支付，冻结，透支，转赠，查询账户余额，查询交易流水等操作。

在这里，我们假定要去实现一个具备充值，提现, 支付，查询余额，还有查询交易流水五个功能的钱包。

其业务流程分别为：

+ 充值 
    + 用户通过第三方支付渠道，将自己银行卡里面的钱充值到虚拟钱包账号当中
    + 操作流程
        + 从用户银行卡到应用的公共银行卡
        + 用户虚拟钱包增加金额
        + 记录刚刚这笔交易流水
+ 支付
    + 实际上是一个转账的过程，从用户的虚拟钱包账户划钱到商家的虚拟钱包账户当中
    + 记录流水信息
+ 提现
    + 用户虚拟钱包  减去对应的钱数
    + 应用的公共银行卡 打钱 到用户的银行卡
    + 记录交易
+ 查询余额
    + 看虚拟钱包的余额数字
+ 查询交易流水
    + 查询充值，支付，提现三种操作


## 3.2 设计思路

首先我们需要对系统进行解耦，即用相似特征和特性的功能放到同一个子系统当中。根据特性，我们可以分为虚拟钱包系统和三方支付系统两个部分。

+ 虚拟钱包
    + 用户虚拟钱包
    + 商家虚拟钱包
+ 三方支付
    + 用户银行卡
    + 商家银行卡
    + 应用公共银行卡

虚拟钱包需要支持的操作基本上就是对于余额的加减，充值，提现，查询三种操作都是只涉及到一个账户的余额的加减操作；而支付功能涉及到两个账户的余额的加减操作。

而对于交易记录，应当记录的信息有：
+ 交易流水ID
+ 交易时间
+ 交易金额
+ 交易类型
    + 充值
    + 提现
    + 支付
+ 入账钱包账号
+ 出账钱包账号

这么设计是有点浪费存储空间的，因为对于充值提现这种交易类型来说，我们只要记录一个钱包账户信息就好了。

另外一种方式就是在交易类型处，设计成支付和被支付两种类型，这样在对待转账的情况的时候，数据库写两条数据，来记录整个transaction。能够省空间，但是会有一些问题：

最重要的难点还是在数据的一致性方面，当我们在做转账操作的时候，我们必须保证加减两个操作要么都成功，要么都失败。如果一个成功，一个失败，那会完蛋的。关于钱的事情，发生一点错误就会对公司造成非常大的影响。

对于转账及类似的操作，合理的做法是在操作两个钱包的账户余额之前，先记录交易流水，并且标记为待执行，当两个钱包的加减金额都完成了之后，我们再回头将交易记录的状态标记为失败。然后我们通过后台的补漏job，拉取状态为失败或者长时间处于待执行状态的交易记录，重新执行或者人工介入处理。

另外一个点在我们会构建一个钱包系统，然后分出两个子系统，虚拟钱包还有第三方交易平台，那么我们的商业逻辑都应该放到钱包系统这一个层级上，我们希望我们的虚拟钱包还有交易平台尽量和我们的商业逻辑脱钩，更多的是事务上方法上的更泛化的东西。这样做的好处是我们的商业逻辑会经常发生变化，但是我们希望虚拟钱包，还有第三方交易平台两个模块不需要经常性的变动。这也是去做两个子系统的初衷之一。

## 3.3 基于贫血模式的传统开发模式



    public class VirtualWalletController {
      // 通过构造函数或者IOC框架注入
      private VirtualWalletService virtualWalletService;
      
      public BigDecimal getBalance(Long walletId) { ... } //查询余额
      public void debit(Long walletId, BigDecimal amount) { ... } //出账
      public void credit(Long walletId, BigDecimal amount) { ... } //入账
      public void transfer(Long fromWalletId, Long toWalletId, BigDecimal amount) { ...} //转账
    }
    
    
    public class VirtualWalletBo {//省略getter/setter/constructor方法
      private Long id;
      private Long createTime;
      private BigDecimal balance;
    }
    
    public class VirtualWalletService {
      // 通过构造函数或者IOC框架注入
      private VirtualWalletRepository walletRepo;
      private VirtualWalletTransactionRepository transactionRepo;
      
      public VirtualWalletBo getVirtualWallet(Long walletId) {
        VirtualWalletEntity walletEntity = walletRepo.getWalletEntity(walletId);
        VirtualWalletBo walletBo = convert(walletEntity);
        return walletBo;
      }
      
      public BigDecimal getBalance(Long walletId) {
        return walletRepo.getBalance(walletId);
      }
      
      public void debit(Long walletId, BigDecimal amount) {
        VirtualWalletEntity walletEntity = walletRepo.getWalletEntity(walletId);
        BigDecimal balance = walletEntity.getBalance();
        if (balance.compareTo(amount) < 0) {
          throw new NoSufficientBalanceException(...);
        }
        walletRepo.updateBalance(walletId, balance.subtract(amount));
      }
      
      public void credit(Long walletId, BigDecimal amount) {
        VirtualWalletEntity walletEntity = walletRepo.getWalletEntity(walletId);
        BigDecimal balance = walletEntity.getBalance();
        walletRepo.updateBalance(walletId, balance.add(amount));
      }
      
      public void transfer(Long fromWalletId, Long toWalletId, BigDecimal amount) {
        VirtualWalletTransactionEntity transactionEntity = new VirtualWalletTransactionEntity();
        transactionEntity.setAmount(amount);
        transactionEntity.setCreateTime(System.currentTimeMillis());
        transactionEntity.setFromWalletId(fromWalletId);
        transactionEntity.setToWalletId(toWalletId);
        transactionEntity.setStatus(Status.TO_BE_EXECUTED);
        Long transactionId = transactionRepo.saveTransaction(transactionEntity);
        try {
          debit(fromWalletId, amount);
          credit(toWalletId, amount);
        } catch (InsufficientBalanceException e) {
          transactionRepo.updateStatus(transactionId, Status.CLOSED);
          ...rethrow exception e...
        } catch (Exception e) {
          transactionRepo.updateStatus(transactionId, Status.FAILED);
          ...rethrow exception e...
        }
        transactionRepo.updateStatus(transactionId, Status.EXECUTED);
      }
    }
    
## 3.4 基于充血模式的DDD开发模式


    public class VirtualWallet { // Domain领域模型(充血模型)
      private Long id;
      private Long createTime = System.currentTimeMillis();;
      private BigDecimal balance = BigDecimal.ZERO;
      
      public VirtualWallet(Long preAllocatedId) {
        this.id = preAllocatedId;
      }
      
      public BigDecimal balance() {
        return this.balance;
      }
      
      public void debit(BigDecimal amount) {
        if (this.balance.compareTo(amount) < 0) {
          throw new InsufficientBalanceException(...);
        }
        this.balance.subtract(amount);
      }
      
      public void credit(BigDecimal amount) {
        if (amount.compareTo(BigDecimal.ZERO) < 0) {
          throw new InvalidAmountException(...);
        }
        this.balance.add(amount);
      }
    }
    
    public class VirtualWalletService {
      // 通过构造函数或者IOC框架注入
      private VirtualWalletRepository walletRepo;
      private VirtualWalletTransactionRepository transactionRepo;
      
      public VirtualWallet getVirtualWallet(Long walletId) {
        VirtualWalletEntity walletEntity = walletRepo.getWalletEntity(walletId);
        VirtualWallet wallet = convert(walletEntity);
        return wallet;
      }
      
      public BigDecimal getBalance(Long walletId) {
        return walletRepo.getBalance(walletId);
      }
      
      public void debit(Long walletId, BigDecimal amount) {
        VirtualWalletEntity walletEntity = walletRepo.getWalletEntity(walletId);
        VirtualWallet wallet = convert(walletEntity);
        wallet.debit(amount);
        walletRepo.updateBalance(walletId, wallet.balance());
      }
      
      public void credit(Long walletId, BigDecimal amount) {
        VirtualWalletEntity walletEntity = walletRepo.getWalletEntity(walletId);
        VirtualWallet wallet = convert(walletEntity);
        wallet.credit(amount);
        walletRepo.updateBalance(walletId, wallet.balance());
      }
      
      public void transfer(Long fromWalletId, Long toWalletId, BigDecimal amount) {
        //...跟基于贫血模型的传统开发模式的代码一样...
      }
    }


一些思考： 
+ 领域模型 希望其尽可能的独立，不包含任何其它层的代码，将流程性的代码逻辑和领域模型的业务逻辑解耦，让领域模型更加可以复用
+ Service类负责一些非功能性的和与第三方交互的工作 
    + 信息传送
    + metrics
    + 日志

# 4. 实战 - 接口鉴权

目的是熟悉在拿到相对笼统的开发需求的时候，需要如何做需求分析，如何做职责划分，看需要定义哪些类，每个类应该具有哪些属性，方法；定义类和类的交互

## 4.1 需求

微服务系统，通过HTTP协议暴露接口给其他系统调用。需要实现一个接口鉴权系统，只有经过认证的系统才能调用我们的接口

+ 需求分析
    + 基础分析
        + 通过用户名加密码来做认证
        + 每个允许访问的调用发都有应用ID还有秘钥，在做接口请求的时候，需要传进来应用ID和秘钥，然后我们在自己的服务器来进行验证比对。如果一致，说明认证成功，允许接口调用了否则，就拒绝
    + 二轮分析
        + 这种方式，明文传输，容易被拦截，并不安全
        + 借助加密算法，对密码进行加密再传递到微服务端验证，同样不安全。因为还是可以被拦截，被拦截以后黑客可以直接拿着这个加密的密码加ID来假装是调用者向服务端发出请求
        + OAuth方式
            + 调用方生成token (id + appId + pwd)
            + 调用方生成新的URL (id + appId + token)
            + Server解析出URL, appId, token
            + Server从数据库根据appId拿出pwd
            + Server利用Url，appId， pwd生成server端token
            + 比较是否一致
    + 三轮分析
        + 上述方式还是可能存在重放攻击，被拦截，然后来伪装成认证系统，调用这个URL对应的接口。
        + token生成过程加入时间戳，然后传递到微服务器端
        + 微服务器收到这些数据之后，会验证当前时间戳跟传递过来的时间戳，是否在一定的时间窗口内。超过时间窗口，也会决绝请求
    + 四轮分析
        + 基本就是到这个程度，因为我们还要考虑性能方面的东西。这种方式对于性能的影响比较小，也考量到了安全性。
        + 如何在微服务端存储每个授权调用方的appId和密码
            + 开发鉴权这种非业务功能，最好不要与具体的第三方系统有过度的耦合
            + 最好能够支持多种不同的存储方式
                + ZooKeeper
                + 本地配置文件
                + 自研配置中心
                + MySQL
                + Redis等
    + 最终需求的确定
        + 调用方进行接口请求的时候，将 URL、AppID、密码、时间戳拼接在一起，通过加密算法生成 token，并且将 token、AppID、时间戳拼接在 URL 中，一并发送到微服务端。
        + 微服务端在接收到调用方的接口请求之后，从请求中拆解出 token、AppID、时间戳。
        + 微服务端首先检查传递过来的时间戳跟当前时间，是否在 token 失效时间窗口内。如果已经超过失效时间，那就算接口调用鉴权失败，拒绝接口调用请求。
        + 如果 token 验证没有过期失效，微服务端再从自己的存储中，取出 AppID 对应的密码，通过同样的 token 生成算法，生成另外一个 token，与调用方传递过来的 token 进行匹配；如果一致，则鉴权成功，允许接口调用，否则就拒绝接口调用。 

## 4.2 面向对象设计

+ 进行职责划分，进而识别出都有哪些类
    + 将需求描述中的名词罗列出来，作为可能的候选类，然后进行筛选
    + 或者根据需求描述，将其中涉及的功能点，一个一个罗列出来，然后再看哪些功能点职责相近，操作同样的属性，能否归到同一个类当中
+ 定义类，及其属性和方法
+ 定义类和类之间的交互关系
+ 将类组装起来并提供执行入口 


+ 功能点列表
    + 把 URL、AppID、密码、时间戳拼接为一个字符串；
    + 对字符串通过加密算法加密生成 token；
    + 将 token、AppID、时间戳拼接到 URL 中，形成新的 URL；
    + 解析 URL，得到 token、AppID、时间戳等信息；
    + 从存储中取出 AppID 和对应的密码；
    + 根据时间戳判断 token 是否过期失效；
    + 验证两个 token 是否匹配； 

从上面的功能列表中，我们发现，1、2、6、7 都是跟 token 有关，负责 token 的生成、验证；3、4 都是在处理 URL，负责 URL 的拼接、解析；5 是操作 AppID 和密码，负责从存储中读取 AppID 和密码。所以，我们可以粗略地得到三个核心的类：AuthToken、Url、CredentialStorage。AuthToken 负责实现 1、2、6、7 这四个操作；Url 负责 3、4 两个操作；CredentialStorage 负责 5 这个操作。


    // AuthToken类的实现
    private static final long DEFAULT_EXPIRED_TIME_INTERVAL = 1 * 60 * 1000;
    private String token;
    private long createTime;
    private long expiredTimeInterval = DEFAULT_EXPIRED_TIME_INTERVAL;
    
    public AuthToken(String token, long createTime);
    public AuthToken(String token, long createTime, long expredTImeInterval);
    
    public static AuthToken create(String baseUrl, long createTime, Map<String, String> params);
    
    public String getToken();
    
    public boolean isExpired();
    
    public boolean match(AuthToken authToken)

+ Tips
    + 并不是所有的需要的名词类的属性都会作为类的属性，有可能会作为方法的参数。选择的基准还是这个属性到底属不属于这个类，从这个角度来看的
    + 我们有可能需要去挖掘一下在功能需求里面并没有体现的一些属性  还是需要从业务模型的角度上来看究竟需要怎么做才比较好



    public interface ApiAuthenticator {
      void auth(String url);
      void auth(ApiRequest apiRequest);
    }
    
    public class DefaultApiAuthenticatorImpl implements ApiAuthenticator {
      private CredentialStorage credentialStorage;
      
      public DefaultApiAuthenticator() {
        this.credentialStorage = new MysqlCredentialStorage();
      }
      
      public DefaultApiAuthenticator(CredentialStorage credentialStorage) {
        this.credentialStorage = credentialStorage;
      }
    
      @Override
      public void auth(String url) {
        ApiRequest apiRequest = ApiRequest.buildFromUrl(url);
        auth(apiRequest);
      }
    
      @Override
      public void auth(ApiRequest apiRequest) {
        String appId = apiRequest.getAppId();
        String token = apiRequest.getToken();
        long timestamp = apiRequest.getTimestamp();
        String originalUrl = apiRequest.getOriginalUrl();
    
        AuthToken clientAuthToken = new AuthToken(token, timestamp);
        if (clientAuthToken.isExpired()) {
          throw new RuntimeException("Token is expired.");
        }
    
        String password = credentialStorage.getPasswordByAppId(appId);
        AuthToken serverAuthToken = AuthToken.generate(originalUrl, appId, password, timestamp);
        if (!serverAuthToken.match(clientAuthToken)) {
          throw new RuntimeException("Token verfication failed.");
        }
      }
    }