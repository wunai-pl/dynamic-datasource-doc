# 本地事务

## 背景

多数据源事务方案一直是一个难题，通常的解决方案有二种。

1. 利用atomiks手动构建多数据源事务，适合数据源较少，配置的参数也不太多的项目。难点就是手动配置量大，需要耗费一定时间。
2. 用seata类似的分布式事务解决方案,难点就是需要搭建维护如seata-server的统一管理中心。

每一种方案都有其适用场景，本项目作者常常收到的问题就是。
1. 为什么我加了事务注解，切换数据源失败？ 
2. 我了解涉及了分布式事务了，但我不想用seata。我场景简单，有没有不依赖第三方的方案？

## 基础介绍

自从3.3.0开始，由seata的核心贡献者https://github.com/a364176773 贡献了基于connection代理的方案。

初版肯定会有一些问题，建议大家在本地仔细测试后再上生产，希望大家多提意见。

## 注意事项

1. 不支持spring原生事务，不支持spring事务，不支持spring事务，可分别使用，千万不能混用。
2. 再次强调不支持spring事务注解，可理解成独立写了一套事务方案。
3. 只适合简单本地多数据源场景， 如果涉及异步和微服务等场景，请使用seata方案。

## 使用方法

完整示例项目 https://github.com/dynamic-datasource/dynamic-datasource-samples/tree/master/tx-local-sample

示例项目用的是和seata方案同一套代码，只是注解换了。

在需要切换数据源且需要事务支持的方法上加@DSTransactional.
```java
@Slf4j
@Service
public class AccountServiceImpl implements AccountService {

    @DS("account")
    @DSTransactional
    public void reduceBalance(Long userId, Double price) {
        log.info("=============ACCOUNT START=================");
        log.info("当前 XID: {}", TransactionContext.getXID());

        Account account = accountMapper.selectById(userId);
        Assert.notNull(account, "用户不存在");
        Double balance = account.getBalance();
        log.info("下单用户{}余额为 {},商品总价为{}", userId, balance, price);

        if (balance < price) {
            log.warn("用户 {} 余额不足，当前余额:{}", userId, balance);
            throw new RuntimeException("余额不足");
        }
        log.info("开始扣减用户 {} 余额", userId);
        double currentBalance = account.getBalance() - price;
        account.setBalance(currentBalance);
        accountMapper.updateById(account);
        log.info("扣减用户 {} 余额成功,扣减后用户账户余额为{}", userId, currentBalance);
        log.info("=============ACCOUNT END=================");
    }
}
```



