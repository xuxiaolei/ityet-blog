---
layout: post
title: MybatisPlus多数据源分页
categories: Java
description: MybatisPlus多数据源分页
index_img: /img/post_def.png
date:  2021-09-07 09:09:09
tags: [MybatisPlus,Java]
---
## 一、MybatisPlusInterceptor


> 从Mybatis Plus 3.4.0版本开始，不再使用旧版本的PaginationInterceptor ，而是使用MybatisPlusInterceptor。



MybatisPlusInterceptor是一系列的实现InnerInterceptor的拦截器链，也可以理解为一个集合。可以包括如下的一些拦截器


- 自动分页: PaginationInnerInterceptor（最常用）
- 多租户: TenantLineInnerInterceptor
- 动态表名: DynamicTableNameInnerInterceptor
- 乐观锁: OptimisticLockerInnerInterceptor
- sql性能规范: IllegalSQLInnerInterceptor
- 防止全表更新与删除: BlockAttackInnerInterceptor



## 二、旧版分页插件配置方法（Mybatis Plus 3.4.0版本之前）


```
@Configuration
@MapperScan(basePackages = {"com.zimug.**.mapper"})
public class MybatisPlusConfig {

    @Bean
    public PaginationInterceptor paginationInterceptor() {
        PaginationInterceptor paginationInterceptor = new PaginationInterceptor();
        // 设置请求的页面大于最大页后操作， true调回到首页，false 继续请求  默认false
        // paginationInterceptor.setOverflow(false);
        // 设置最大单页限制数量，默认 500 条，-1 不受限制
        // paginationInterceptor.setLimit(500);
        // 开启 count 的 join 优化,只针对部分 left join
        paginationInterceptor.setCountSqlParser(new JsqlParserCountOptimize(true));
        return paginationInterceptor;
    }
}
```


## 三、新的分页插件配置方法（Mybatis Plus 3.4.0版本及其之后的版本）


新的分页插件,一缓和二缓遵循mybatis的规则,需要设置 MybatisConfiguration#useDeprecatedExecutor = false 避免缓存出现问题


```
@Configuration
@MapperScan(basePackages = {"com.zimug.**.mapper"})
public class MybatisPlusConfig {

  /**
   * 新的分页插件,一缓和二缓遵循mybatis的规则,需要设置 MybatisConfiguration#useDeprecatedExecutor = false 避免缓存出现问题(该属性会在旧插件移除后一同移除)
   */
  @Bean
  public MybatisPlusInterceptor mybatisPlusInterceptor() {
    MybatisPlusInterceptor interceptor = new MybatisPlusInterceptor();
    //向Mybatis过滤器链中添加分页拦截器
    interceptor.addInnerInterceptor(new PaginationInnerInterceptor(DbType.MYSQL));
    //还可以添加i他的拦截器
    return interceptor;
  }

  @Bean
  public ConfigurationCustomizer configurationCustomizer() {
    return configuration -> configuration.setUseDeprecatedExecutor(false);
  }
}
```


## 四、分页查询的使用方法


**分页查询的使用方法没有变化，仍然和Mybatis之前的版本一致，没有变化**。 这里简单举一个例子


```
Page<SysUserOrg> page = new Page<> (pageNum,pageSize);   //查询第pageNum页，每页pageSize条数据
//将分页参数page作为Mybatis或Mybatis Plus的第一个参数传入持久层函数，即可完成分页查询
return mySystemMapper.selectUser(page, 其他参数 );
```

## 五、shardingSphere+多数据源情况下配置方法
```java
@Configuration
@MapperScan(basePackages = {"com.jkys.operate.equity.dal.mapper"}, sqlSessionFactoryRef = "sqlSessionFactoryVip")
public class DbConfigVip {
    @Bean("sqlSessionFactoryVip")
    public SqlSessionFactory sqlSessionFactoryVip(@Qualifier("dataSourceVip") DataSource dataSource) throws Exception {
        MybatisSqlSessionFactoryBean sqlSessionFactory = new MybatisSqlSessionFactoryBean();
        sqlSessionFactory.setDataSource(dataSource);
        MybatisConfiguration configuration = new MybatisConfiguration();
        configuration.setDefaultScriptingLanguage(MybatisXMLLanguageDriver.class);
        configuration.setJdbcTypeForNull(JdbcType.NULL);
        sqlSessionFactory.setConfiguration(configuration);
        sqlSessionFactory.setMapperLocations(new PathMatchingResourcePatternResolver().
                getResources("classpath*:mapper/**/*.xml"));
        //载入插件
        sqlSessionFactory.setPlugins(paginationInterceptor());
        sqlSessionFactory.setGlobalConfig(new GlobalConfig().setBanner(false));
        return sqlSessionFactory.getObject();
    }

    /**
     * 分页插件
     */
    @Bean
    public MybatisPlusInterceptor paginationInterceptor() {
        MybatisPlusInterceptor paginationInterceptor = new MybatisPlusInterceptor();
        paginationInterceptor.addInnerInterceptor(new PaginationInnerInterceptor(DbType.MYSQL));
        return paginationInterceptor;
    }

    @Primary
    @Bean(name = "transactionManagerVip")
    public DataSourceTransactionManager transactionManagerVip(@Qualifier("dataSourceVip") DataSource dataSource) {
        return new DataSourceTransactionManager(dataSource);
    }

    @Primary
    @Bean(name = "sqlSessionTemplateVip")
    public SqlSessionTemplate sqlSessionTemplateVip(@Qualifier("sqlSessionFactoryVip") SqlSessionFactory sqlSessionFactory) {
        return new SqlSessionTemplate(sqlSessionFactory);
    }
}
```

