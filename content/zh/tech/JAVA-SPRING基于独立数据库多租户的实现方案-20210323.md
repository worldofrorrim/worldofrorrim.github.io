+++
title = "多租户实现方案"
date = "2021-03-23T20:56:00+08:00"
tags = ["多租户","JAVA", "SPRING"]
slug = "多租户的实现方案"
indent = true

+++

# 多租户的实现方案


## 1.  常见的多租户实现方案

目前基于多租户的数据库设计方案通常有如下三种：
### 1.1 独立数据库

针对独立数据库的这种方式，首先需要业务层能够支持多数据源的配置，并且为每个租户创建或初始化一个数据库。应用程序和数据库都是独立的实例，因此它不会与任何其他独立实例交互。只为一个租户提供服务，拥有独立的服务、独立的数据库以及独立的请求处理。

![image](C:\Users\jiaoj\Desktop\current\worldofrorrim\static\images\15267)
独立数据库：每个租户一个数据库。

**优点**：为不同的租户提供独立的数据库，有助于简化数据模型的扩展设计，满足不同租户的独特需求；如果 出现故障，恢复数据比较简单；

**缺点**： 增多了数据库的安装数量，随之带来维护成本和购置成本的增加

**这种方案与传统的一个客户、一套数据、一套部署类似，差别只在于软件统一部署在运营商那里。由此可见此方案用户数据隔离级别高，安全性好，但是成本较高**

### 1.2 独立 Schema 共享数据库

共享数据库、独立Schema模式，是将多个或所有租户的数据放在一个数据库服务中，但是为每一个租户建立一个独立的schema。租户间数据彼此逻辑不可见，上层应用程序的实现和独立数据库一样简单。（补充：mysql数据中的schema比较特殊，并不是数据库的下一级，而是等同于数据库。）

**优点**：对于安全性要求较高的租户，是一种选择。提供了一定程度的逻辑数据隔离，但并不是完全隔离；每个数据库可支持更多的租户数量。

**缺点**：如果出现故障，数据恢复比较困难，因为恢复数据库将牵涉到其他租户的数据；如果需要跨租户统计数据，存在一定困难。这种方案是方案一的变种。只需要安装一份数据库服务，通过不同的Schema对不同租户的数据进行隔离。由于数据库服务是共享的，所以成本相对低廉

**这种方案是方案一的变种。只需要安装一份数据库服务，通过不同的Schema对不同租户的数据进行隔离。由于数据库服务是共享的，所以成本相对低廉。**

### 1.3. 共享数据库且共享数据表

共享数据库、共享数据表：即租户共享同一个Database，同一套数据库表（所有租户的数据都存放在一个数据库 的同一套表中）。在表中增加租户ID等租户标志字段，表明该记录是属于哪个租户的
![image](C:\Users\jiaoj\Desktop\current\worldofrorrim\static\images\15265)
**优点**：所有租户使用同一套数据库，所以成本低廉。

**缺点**：隔离级别低，安全性低，需要在设计开发时加大对安全的开发量，数据备份和恢复困难。

**这种方案和基于传统应用的数据库设计并没有任何区别，但是由于所有租户使用相同的数据库表，所以需要做好对每个租户数据的隔离安全性处理，这就增加了系统设计和数据管理方面的复杂程度。**

## 2.  独立数据库 实现方案
独立数据库，即一个租户一个数据库，这种方案的用户数据隔离级别最高，安全性最好，但成本较高。

技术实现架构设计图
![image](C:\Users\jiaoj\Desktop\current\worldofrorrim\static\images\15246)


 实现流程

1. 用户发起请求（请求携带租户信息 tenant）
2. 请求到应用系统，应用系统拦截器得到当前请求的租户信息，动态切换数据和redis
3. 在当前租户的数据库或者redis上面进行业务操作





实现技术难点

1. 实现 数据库 动态数据源
2. 实现 redis 动态数据源
3. 实现 oss 基于租户信息 保存到指定bucket下
4. 实现 mq 对于多租户支持



### 3. msyql 动态数据源 java代码实现
####  3.1 基于spring AbstractRoutingDataSource 实现多数据源动态切换

oss  和 redis 可以参考mysql的实现，进行动态切换

DynamicDataSource  类实现
```java
public class DynamicDataSource extends AbstractRoutingDataSource {
    /**
     * 如果不希望数据源在启动配置时就加载好，可以定制这个方法，从任何你希望的地方读取并返回数据源
     * 比如从数据库、文件、外部接口等读取数据源信息，并最终返回一个DataSource实现类对象即可
     */
    @Override
    protected DataSource determineTargetDataSource() {
        return super.determineTargetDataSource();
    }

    /**
     * 如果希望所有数据源在启动配置时就加载好，这里通过设置数据源Key值来切换数据，定制这个方法
     */
    @Override
    protected Object determineCurrentLookupKey() {
        return DynamicDataSourceContextHolder.getDataSourceKey();
    }

    /**
     * 设置默认数据源
     *
     * @param defaultDataSource
     */
    public void setDefaultDataSource(Object defaultDataSource) {
        super.setDefaultTargetDataSource(defaultDataSource);
    }

    /**
     * 设置数据源
     *
     * @param dataSources
     */
    public void setDataSources(Map<Object, Object> dataSources) {
        super.setTargetDataSources(dataSources);
        // 将数据源的 key 放到数据源上下文的 key 集合中，用于切换时判断数据源是否有效
        DynamicDataSourceContextHolder.addDataSourceKeys(dataSources.keySet());
    }
}
```




 DynamicDataSourceContextHolder  类实现

```
public class DynamicDataSourceContextHolder {
    private static final ThreadLocal<String> contextHolder = new ThreadLocal<String>() {
        /**
         * 将 master 数据源的 key作为默认数据源的 key
         */
        @Override
        protected String initialValue() {
            return "master";
        }
    };
    /**
     * 数据源的 key集合，用于切换时判断数据源是否存在
     */
    public static List<Object> dataSourceKeys = new ArrayList<>();

    /**
     * 获取数据源
     *
     * @return
     */
    public static String getDataSourceKey() {
        return contextHolder.get() == null ? MybatisPlusConfig.DEFAULT_DATASOURCKEY : contextHolder.get();
    }

    /**
     * 切换数据源
     *
     * @param key
     */
    public static void setDataSourceKey(String key) {
        contextHolder.set(key);
    }

    /**
     * 重置数据源
     */
    public static void clearDataSourceKey() {
        contextHolder.remove();
    }

    /**
     * 判断是否包含数据源
     *
     * @param key 数据源key
     * @return
     */
    public static boolean containDataSourceKey(String key) {
        return dataSourceKeys.contains(key);
    }

    /**
     * 添加数据源keys
     *
     * @param keys
     * @return
     */
    public static boolean addDataSourceKeys(Collection<? extends Object> keys) {
        return dataSourceKeys.addAll(keys);
    }
}
```
Mybatis 数据源配置
```java
 @Bean("dynamicDataSource")
    public DataSource dynamicDataSource() {
        DynamicDataSource dynamicDataSource = new DynamicDataSource();
        Map<Object, Object> dataSourceMap = new HashMap<>();
        List<DynamicDataSourceProperty.DynamicTenantDataSourceProperty> datasources = dynamicDataSourceProperty.getDatasources();
        String defaultTennatId = null;
        if (!datasources.isEmpty()) {
            for (DynamicDataSourceProperty.DynamicTenantDataSourceProperty dataSourceProperty :
                    datasources) {
                String tenantname = dataSourceProperty.getTenantname();
                String tenantId = dataSourceProperty.getTenantId();
                Boolean isDefault = dataSourceProperty.getIsDefault();
                if (isDefault != null && isDefault) {
                    DEFAULT_TENANTID = tenantId;
                    DEFAULT_DATASOURCKEY = DEFAULT_TENANTID + DATASOURCKEY_SEPARATOR + DEFAULT_DATASOURCENAME;
                }

                List<DynamicDataSourceProperty.Source> sources = dataSourceProperty.getSources();
                for (DynamicDataSourceProperty.Source source : sources) {
                    HikariDataSource dataSource = new HikariDataSource();
                    String driverClassName = source.getDriverClassName();
                    String jdbcUrl = source.getJdbcUrl();
                    String username = source.getUsername();
                    String password = source.getPassword();
                    String sourceName = source.getName();

                    dataSource.setDriverClassName(driverClassName);
                    dataSource.setJdbcUrl(jdbcUrl);
                    dataSource.setUsername(username);
                    dataSource.setPassword(password);
                    log.info(" datasource  key is " + tenantId + DATASOURCKEY_SEPARATOR + sourceName + "，name is {}", tenantname);
                    dataSourceMap.put(tenantId + DATASOURCKEY_SEPARATOR + sourceName, dataSource);

                    TENANTLIST.add(Tenant.builder().tenantId(tenantId).tenantName(tenantname).isDefault(isDefault).build());
                }

            }
        }

        log.info("datasource size : {}", dataSourceMap.size());
        DataSource defaultDataSource = (DataSource) dataSourceMap.get(DEFAULT_DATASOURCKEY);
        // 将 master 数据源作为默认指定的数据源
        dynamicDataSource.setDefaultDataSource(defaultDataSource);
        // 将 master 和 slave 数据源作为指定的数据源
        dynamicDataSource.setDataSources(dataSourceMap);
        return dynamicDataSource;
    }
```

### 4 MQ 在多租户环境的处理

1. 通过MQ 数据区分来表示来自于哪个租户，处理数据的时候 根据租户的id切换到指定数据库进行业务处理，或者切换到指定的redis进行处理

```

@Service
public class LoginMessageConsumerListener implements ChannelAwareMessageListener {

    @Autowired
    SysAccountService accountService;

    @Override
    public void onMessage(Message message, Channel channel) throws Exception {
        byte[] body = message.getBody();
        System.out.println(Thread.currentThread().getName() + " login 收到消息：" + new String(body, Charset.forName("utf-8")));
        //获取租户信息
        String s = new String(body, Charset.forName("utf-8"));
        Map<String, Object> map = JsonsUtil.toMap(s);
        String tenant = (String) map.get("tenant");
        String data = (String) map.get("data");

        //切换数据源
        TenantContextHolder.setCurrentTenant(tenant);
        DynamicDataSourceContextHolder.setDataSourceKey(tenant + "-master");


        //做一些 业务操作
        SysAccount user = accountService.getOne(Wrappers.<SysAccount>lambdaQuery().eq(SysAccount::getUsername, "test"));
        System.out.println("用户名：" + user.getUsername() + ", 用户id:" + user.getAccountId().toString());

        //ack
        channel.basicAck(message.getMessageProperties().getDeliveryTag(), false);

    }
}
```





## 5 统一用户多租户改造遇到的问题
- 1. 页面跳转需要手动传递租户id
由于统一用户采用 header 携带 租户信息给后台接口，有些情况下前端不能携带header信息
比如用户单点登录  跳转到另一个系统，header无法携带租户id，那么就需要手动传递租户信息 ,若采用基于二级域名做切换（泛域名解析）则不存在此问题。

-  2. 定时任务需要遍历所有的租户数据，每个租户数据库执行一次


-  3.  多线程环境或者线程池 中需要手动设置当前租户是谁，才能切换到指定数据库


-  4.  提供外部接口，调用接口需要指定请求的租户是谁，否则调用的是默认租户


-     5.跨租户进行数据统计，比如统计所有租户的用户数量等，需要一个数据库一个数据库的统计

- 6. 性能方面，由于每次处理请求需要切换指定租户的数据源，性能上会有一定的损耗， 在租户多的情况下待整体性能下降

- 7.  技术上 是基于spingboot的 AbstractRoutingDataSource，所以得依赖spring

