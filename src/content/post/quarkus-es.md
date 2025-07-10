---
title: "Quarkus 读取ES数据"
description: "使用 `hibernate-search-standalone-elasticsearch` 扩展来读取 Elasticsearch 数据"
publishDate: "2025-07-10"
tags: ["qarkus", "es", "spring"]
---

> 过程参考[Quarkus官方指南](https://quarkus.io/guides/hibernate-search-standalone-elasticsearch)

# Quarkus 使用 Hibernate Search Standalone Elasticsearch 读取 ES 数据

本文将介绍如何在 Quarkus 中使用 `hibernate-search-standalone-elasticsearch` 扩展来读取 Elasticsearch 数据。

## 环境准备

首先需要添加必要的依赖：

```shell
./gradlew addExtension --extensions='hibernate-search-standalone-elasticsearch'
```

## 实现步骤

### 1. 创建实体类

首先定义一个映射到 Elasticsearch 索引的实体类：

```java
package cn.jl.zzq.entity.es;

import org.hibernate.search.engine.backend.types.Sortable;
import org.hibernate.search.mapper.pojo.mapping.definition.annotation.FullTextField;
import org.hibernate.search.mapper.pojo.mapping.definition.annotation.GenericField;
import org.hibernate.search.mapper.pojo.mapping.definition.annotation.Indexed;
import org.hibernate.search.mapper.pojo.mapping.definition.annotation.KeywordField;

@Indexed
public class SocialPostEntity {
    
    @KeywordField
    private String id;
    
    @FullTextField(analyzer = "standard")
    private String content;
    
    @KeywordField
    private String accountId;
    
    @FullTextField(analyzer = "standard")
    private String accountName;
    
    @KeywordField
    private String postType;
    
    @KeywordField
    private String platform;
    
    @GenericField(sortable = Sortable.YES)
    private Long publishTime;
    
    // 省略getter和setter方法
}
```

### 2. 创建服务类

实现搜索逻辑的服务类：

```java
package cn.jl.zzq.service;

import cn.jl.zzq.entity.es.SocialPostEntity;
import jakarta.enterprise.context.ApplicationScoped;
import jakarta.inject.Inject;
import org.hibernate.search.engine.search.predicate.dsl.BooleanPredicateClausesStep;
import org.hibernate.search.engine.search.predicate.dsl.PredicateFinalStep;
import org.hibernate.search.engine.search.predicate.dsl.SearchPredicateFactory;
import org.hibernate.search.mapper.pojo.standalone.mapping.SearchMapping;

import java.util.List;
import java.util.Optional;

@ApplicationScoped
public class SocialPostService {

    @Inject
    SearchMapping searchMapping;

    /**
     * 发文搜索
     *
     * @param keyword     搜索关键字，支持 content 字段的模糊搜索
     * @param accountId   账户ID，精确匹配
     * @param accountName 账户名称，模糊匹配
     * @param postType    发文类型，精确匹配
     * @param platform    平台，支持多值
     * @param startTime   起始时间
     * @param endTime     结束时间
     * @param size        最大返回数量，默认为 100
     * @return 搜索结果
     */
    public List<SocialPostEntity> searchSocialPosts(
            Optional<String> keyword,
            Optional<String> accountId,
            Optional<String> accountName,
            Optional<String> postType,
            Optional<List<String>> platform,
            Optional<Long> startTime,
            Optional<Long> endTime,
            Optional<Integer> size
    ) {
        try (var searchSession = searchMapping.createSession()) {
            return searchSession.search(SocialPostEntity.class)
                    .where(f -> buildPredicate(f, keyword, accountId, accountName, 
                            postType, platform, startTime, endTime))
                    .sort(f -> f.field("publishTime").desc())
                    .fetch(size.orElse(100))
                    .hits();
        }
    }

    private PredicateFinalStep buildPredicate(
            SearchPredicateFactory f,
            Optional<String> keyword,
            Optional<String> accountId,
            Optional<String> accountName,
            Optional<String> postType,
            Optional<List<String>> platform,
            Optional<Long> startTime,
            Optional<Long> endTime
    ) {
        BooleanPredicateClausesStep<?, ?> bool = f.bool();

        keyword.ifPresent(k -> bool.must(f.match()
                .fields("content", "accountName")
                .matching(k)
                .fuzzy(1, 2)));

        accountId.ifPresent(id -> bool.must(f.match()
                .field("accountId")
                .matching(id)));

        accountName.ifPresent(name -> bool.must(f.match()
                .field("accountName")
                .matching(name)
                .fuzzy()));

        postType.ifPresent(type -> bool.must(f.match()
                .field("postType")
                .matching(type)));

        platform.ifPresent(platforms -> {
            if (!platforms.isEmpty()) {
                bool.must(f.terms()
                        .field("platform")
                        .matchingAny(platforms));
            }
        });

        if (startTime.isPresent() || endTime.isPresent()) {
            bool.must(f.range()
                    .field("publishTime")
                    .between(startTime.orElse(0L), 
                            endTime.orElse(Long.MAX_VALUE)));
        }

        return bool;
    }
}
```

### 3. 创建 REST 接口

提供搜索接口的 REST 资源类：

```java
package cn.jl.zzq.controller.resource;

import cn.jl.zzq.entity.es.SocialPostEntity;
import cn.jl.zzq.service.SocialPostService;
import jakarta.inject.Inject;
import jakarta.ws.rs.GET;
import jakarta.ws.rs.Path;
import org.eclipse.microprofile.openapi.annotations.Operation;
import org.eclipse.microprofile.openapi.annotations.parameters.Parameter;
import org.eclipse.microprofile.openapi.annotations.tags.Tag;
import org.jboss.resteasy.reactive.RestQuery;

import java.util.List;
import java.util.Optional;

@Path("/post")
@Tag(name = "SocialPost", description = "提供社交平台的发文相关接口")
public class SocialPostResource {

    @Inject
    SocialPostService socialPostService;

    @GET
    @Path("/search")
    @Operation(summary = "搜索发文", description = "根据关键词搜索发文信息")
    public List<SocialPostEntity> searchPosts(
            @RestQuery
            @Parameter(description = "关键词")
            Optional<String> keyword,

            @RestQuery("account_id")
            @Parameter(description = "账号ID")
            Optional<String> accountId,

            @RestQuery("account_name")
            @Parameter(description = "账号名称")
            Optional<String> accountName,

            @RestQuery("post_type")
            @Parameter(description = "发文类型")
            Optional<String> postType,

            @RestQuery
            @Parameter(description = "平台")
            Optional<List<String>> platform,

            @RestQuery("start_time")
            @Parameter(description = "起始时间")
            Optional<Long> startTime,

            @RestQuery("end_time")
            @Parameter(description = "结束时间")
            Optional<Long> endTime,

            @RestQuery
            @Parameter(description = "返回数量限制")
            Optional<Integer> limit
    ) {
        return socialPostService.searchSocialPosts(
                keyword,
                accountId,
                accountName,
                postType,
                platform,
                startTime,
                endTime,
                limit
        );
    }
}
```

## 配置 Elasticsearch 连接

在 `application.properties` 中配置 Elasticsearch 连接信息：

```properties
# Elasticsearch 服务器地址
quarkus.hibernate-search-orm.elasticsearch.hosts=elasticsearch:9200
# 协议 (http 或 https)
quarkus.hibernate-search-orm.elasticsearch.protocol=http
# 用户名 (如果需要认证)
quarkus.hibernate-search-orm.elasticsearch.username=elastic
# 密码 (如果需要认证)
quarkus.hibernate-search-orm.elasticsearch.password=changeme
# 是否自动创建索引
quarkus.hibernate-search-orm.schema-management.strategy=validate
```

## 功能说明

1. **多字段搜索**：支持在内容和账户名称字段中进行模糊搜索
2. **精确匹配**：支持账户ID和发文类型的精确匹配
3. **多值过滤**：支持平台的多值过滤
4. **时间范围**：支持按发布时间范围过滤
5. **结果排序**：默认按发布时间降序排列
6. **分页控制**：支持限制返回结果数量

## 使用示例

可以通过以下方式调用 API：

```
GET /post/search?keyword=测试&account_id=123&platform=weibo,wechat&limit=10
```

## 总结

通过 Quarkus 的 `hibernate-search-standalone-elasticsearch` 扩展，我们可以方便地实现 Elasticsearch 的数据访问功能。这种方式提供了类型安全的查询构建方式，并且与 Quarkus 的无缝集成使得开发效率大大提高。

相比传统的 Elasticsearch 客户端，Hibernate Search 提供了更高级的抽象，使得开发者可以专注于业务逻辑而不是底层查询细节。

:::note
内容由DeepSeek整理
:::