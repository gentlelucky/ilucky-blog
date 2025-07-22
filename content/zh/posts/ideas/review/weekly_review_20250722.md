---
title: "周报 Vol.27 - 消失的成就感"
date: 2025-07-22T16:25:00+08:00
draft: false
tags: 
  - "review"
  - "life"
  - "cashbook"
  - "feel"
  - "code"
  - "design pattern"
categories: 
  - "Ideas"
authors:
  - "gentlelucky"
---

## 前言

本篇是对  `2025-05-19`  到  `2025-07-20`  这段时间生活的记录和思考。

某灵工结算系统规划设计「**内部记账本功能**」，完成了需求分析、架构设计、代码开发、到最终的成功上线，但内心毫无成就感；

## 消失的成就感

### 需求分析

客户希望「结算平台」能够拥有独立的「**内部记账本功能**」，即同一个「主体账号」，系统基于「主体账号」给每一个客户创建独立的「虚拟子账号」；

举例：甲客户给「主体账号」 充值了 **88** 元，乙客户给「主体账号」 充值了 **66** 元，此时「主体账号」金额 **154** 元。「结算平台」要区分出来「主体账号」中 **88** 元属于甲客户的余额，**66** 元属于乙客户的余额。并且支持独立的转账、充值、提现等功能。

由于自己对「**内部记账本**」功能没接触过，所以没有经验。于是参考了一些技术文章，设计了出版的技术概要。

![image-20250721164451653](https://image.gentlelucky.com/image-20250721164451653.png)

![image-20250721164620348](https://image.gentlelucky.com/image-20250721164620348.png)

当刚了解到这个需求后，我做了如下分析：

- 该功能涉及到「钱」的流转，如果控制不好，会导致多发，算重大事故
- 该功能的实现，现有的程序架构已不满足，需重新设计

通过分析，决定如下：

- 预留充足的任务周期：立刻启动「内部记账本」项目，用来多轮测试和验证。并合理的安排任务及任务负责人。
- 核心逻辑：
  - 对接三方结算系统（银行、支付宝...）
  - 内部记账本核心逻辑（线程安全、余额一致性）

### 研发任务

#### 在线文档

使用「**腾讯在线文档**」的智能表格来分配任务。

![image-20250722144215559](https://image.gentlelucky.com/image-20250722144215559.png)

先新建一个「ALL-表格」，把所有任务罗列进去，便于全局查看任务。每个任务尽可能的控制在一人天，这样能够更加细粒度的把控任务进度。当出去任务延期时，也可快速察觉和纠正。

今天-看板：用来查看**当天**有哪些任务需要完成

本周-看板：用来查看**本周**有哪些任务需要完成，周维度查看直观的查看到本周的任务是否有堆积情况

未完成-看板：查看所有过期未完成的任务，次看板任务越多，就需要采取措施：投入更多的时间（加班）、投入更多的人力资源（加人 ）

姓名-看板：个人负责的任务看板，方便个人了解自己研发任务及计划

### 概要设计

![image-20250722142600474](https://image.gentlelucky.com/image-20250722142600474.png)

通过对需求的分析，编写了内部记账本概要设计，主要涵盖**表设计**、**流程说明**、**余额设计**、**充值设计**。

概要设计能够快速拉通与研发个人的认知偏差，统一代码实现逻辑。

### 核心设计

#### 结算通道对接

内部记账本需要对接多结算渠道：招商、支付宝、平安......，并且都会调用各结算渠道的代发、代发状态查询、回执单下载等接口。

针对于各种类型的结算通道，使用枚举类。

```java
@Getter
@AllArgsConstructor
public enum SettlementChannel {
    CMB("CMB","招商银行"),
    PAB("PAB","平安银行"),
    ALI_PAY("ALI_PAY","支付宝");
    private final String code;
    private final String name;
}
```

对于不同结算渠道代发的实现方式不同，定义统一的接口，使用策略模式来获取对应的具体实现。

```java
public interface Transfer {
  
    /**
     * 结算通道
     * @return {@link SettlementChannel}
     */
    SettlementChannel channel();

    /**
     * 代发
     * @param request {@link TransferRequest}
     * @return
     */
    TransferResponse transfer(TransferRequest request);

}
```

```java
@Component
public class TransferFactory implements InitializingBean {

    @Resource
    Collection<Transfer> transfers;
    private Map<SettlementChannel, Transfer> transferMap;

    public Transfer get(SettlementChannel channel) {
        Transfer transfer = transferMap.get(channel);
        if (Objects.isNull(transfer)) {
            return null;
        }
        return transfer;
    }

    @Override
    public void afterPropertiesSet() throws Exception {
        this.transferMap = IterUtil.toMap(transfers, Transfer::channel);
    }
}
```

#### 余额设计

余额存在同一时刻并发操作：支出、充值；所以余额变更需是线程安全的；

```java
public class DefaultAccountBookLock implements AccountBookLock {
    // 账户类
    private volatile CashbookAccountBook cashbookAccountBook;
    // 锁
    private final ReentrantLock lock;
    // 初始化
    public DefaultAccountBookLock(CashbookAccountBook cashbookAccountBook) {
        this.cashbookAccountBook = cashbookAccountBook;
        this.lock = new ReentrantLock();
    }
    // 加锁
    @Override
    public void lock() {
        lock.lock();
    }
    // 释放锁
    @Override
    public void unlock() {
        lock.unlock();
    }
}
```

定义**AccountBookLock**类，核心操作是**ReentrantLock**加锁和释放锁逻辑。一个账号对应一个锁。这样可保证在并发下线程是安全的。

```java
@Slf4j
@Service
public class AccountBookManager implements AccountBook, InitializingBean {
    // 账户锁
    private final ConcurrentHashMap<String, AccountBookLock> accountLocks = new ConcurrentHashMap<>();

    /**
     * 创建账户，账户信息存并发安全的ConcurrentHashMap中
     * @param cashbookAccountBook
     * @return
     */
    @Override
    public void create(CashbookAccountBook cashbookAccountBook) {
        accountLocks.computeIfAbsent(cashbookAccountBook.getAccountBookId(),
                o -> new DefaultAccountBookLock(cashbookAccountBook));
    }

		// 代发
    @Override
    public void payroll(String accountBookId, BigDecimal rechargeAmount, BigDecimal frozenAmount) {
        AccountBookLock accountBookLock = this.getAccountBookLock(accountBookId);
        try {
            accountBookLock.lock();
            log.info(">>> [记账本][余额] 代发开始 | accountBookId={}", accountBookId);
						// 代码逻辑......
            log.info(">>> [记账本][余额] 代发结束 | accountBookId={}", accountBookId);
        } finally {
            accountBookLock.unlock();
        }
    }

    
    private AccountBookLock getAccountBookLock(String accountBookId) {
        AccountBookLock accountBookLock = accountLocks.get(accountBookId);
        if (accountBookLock == null) {
            log.error("[记账本][余额] 账户不存在 | accountBookId={}", accountBookId);
            throw new ServiceException("账户不存在");
        }
        return accountBookLock;
    }

    @Override
    public void afterPropertiesSet() throws Exception {
        // 初始化数据
        for (CashbookAccountBook accountBook : list) {
            this.create(accountBook);
        }
    }

}
```

**AccountBookManager**统一对外操作余额接口，先使用「**accountBookLock.lock()**」方法对方法加锁，业务逻辑执行结束后，在 finally 代码块中释放锁「**accountBookLock.unlock()**」，从而实现了对余额进行**增**或**减**的线程安全操作。

### 回退方案

针对每一次大版本的迭代上线，一定要考虑如何当上线版本出现重大问题，如何快速回退到上一个版本。

- 数据库层面
  - 备份当前数据
  - 尽量不要修改已使用字段含义
- 程序层面
  - 如果条件允许，可使用配置文件切换新旧代码版本
  - 做好上一个版本的程序备份
- 人员层面
  - 提前告知相关人员，如果回退，大家各尽其责

### 成功上线

6 月底完成了研发测试工作，7 月 18 日迭代上线。

当晚上线还算比较顺利，上线后系统稳定运行，未出现结算多发、少发、余额不一致情况。

### 总结反思

上一份周报停留在 **2025-05-18** 那一周，大部分原因是由于「内部记账本」研发周期严重偏离了原计划。导致自己大部分时间都投入到研发工作中，没有太多心思和精力去写博客了。

研发周期严重偏离计划有如下原因：

- 个人出差**半个月**，没精力去统称计划
- 研发对「内部记账本」需求理解偏差，导致返工
- 研发对「余额」变更的逻辑设计存在缺陷

原计划中，我是「项目管理」的角色。但后期又以研发的角色加入项目，编写了：

- 结算通道对接代码（招商、支付宝）
- 余额并发安全代码
- 记账本业务代码
- 测试联调bug修复

从本次项目，我收获到：

1. 当自己有其他事情时，可把当天项目委托给另一个靠谱的人暂时托管
2. 当核心逻辑研发人员提出自己的设计时，存在不合理时，要否定。要不然会导致项目后期，加班去推翻逻辑重写
3. 概要设计要尽可能的详细，这样可减少需求偏差

### 成就感

![bae3bc0cd26bb00a9e484ad1bb79284c](https://image.gentlelucky.com/bae3bc0cd26bb00a9e484ad1bb79284c.jpg)

2016 年 7 月 17 日，完成了「**志邦7·17**」现场抽奖大屏项目。当时成就感满满，那种发自内心的开心和喜悦是无法用言语表达出来的。

2025 年 7 月 18 日，「**内部记账本**」成功上线；内心却毫无波澜，没有任何成就感。即使「**内部记账本**」的技术难度比「**志邦7·17**」高了好几个 Level，也并不会因为技术提升而感到骄傲。

我不知道这种状态是否正常，我也不知道什么样的工作任务，能让我找到之前「成就感」的初心。

## 有趣的事与物

### 输入

虽然大部分有意思的输入会在 「[Lucky’s Footprints](https://t.me/wxluckya)」 Telegram 频道里自动同步，不过还是挑选一部分在这里列举一下。并且把 Telegram Channel 消息作为内容源搭建了一个微博客 —— 「[daily.gentlelucky.com](https://daily.gentlelucky.com/)」，可以更方便浏览了。

#### 收藏

- [The Everything App]([download.anytype.io...](https://download.anytype.io/))，Obsidian平替方案

#### 视频

- [被预制菜包围的一天](https://www.bilibili.com/video/BV1RXgTzxE1w/)
- [便宜没好货啊](https://www.bilibili.com/video/BV1zZGAzWEti/)

#### 音乐

- [III (Find Yourself) - Athletics]([music.163.com...](https://music.163.com/song?id=31370542))
- [回不去的夏天 - 夏日入侵企画]([music.163.com...](https://music.163.com/song?id=1832684671))

#### 文章

- [独立开发出海，需要点哪些技能树 | 辣条的博客]([dev-life.online...](https://dev-life.online/blog/dulikaifa_jineng))

#### 影视

- [哈哈哈哈哈 第五季]([movie.douban.com...](https://movie.douban.com/subject/37054768/))
- [长安的荔枝]([movie.douban.com...](https://movie.douban.com/subject/35651341/))
