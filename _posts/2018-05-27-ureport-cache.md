---
layout: article
title: UReport 的缓存设置
tags: UReport
key: 2018-05-27-ureport-cache
---

# UReport 缓存

## 描述
默认情况下，UReport 有设置 HTTPSession 来缓存报表，并且还有个内置的内存缓存来缓存报表定义。

注意这是两种缓存：

- `com.bstek.ureport.cache.ReportCache` 缓存的内容是报表，包括报表的数据、图表
- `com.bstek.ureport.cache.ReportDefinitionCache` 缓存的是报表的设计

默认情况下，有一个 `com.bstek.ureport.console.cache.HttpSessionReportCache` 来缓存报表；另一个 `com.bstek.ureport.cache.DefaultMemoryReportDefinitionCache` 来缓存报表定义。（不知道为什么不放到一起）

## 出现问题的场景

在少量设计场景中，需要直接从文件或数据库中修改报表定义的 xml 内容来做到一些高级设计，就会发现虽然文件和数据库内容已经改了，但是报表的预览还是原来的，这就是因为报表定义的缓存没有更新。

UReport 只提供了在线正常读、更新缓存，但是没有清除缓存的接口。

## 禁用报表缓存

之前看到有设置

```xml
<bean id="ureport.httpSessionReportCache" class="com.bstek.ureport.console.cache.HttpSessionReportCache">
	<property name="disabled" value="${ureport.disableHttpSessionReportCache}"></property>
</bean>
```

于是可以在 **config.properties** 里面设置 `ureport.disableHttpSessionReportCache=true` 来禁用报表缓存，但是 UReport 有个小 bug（[已修复](https://github.com/youseries/ureport/commit/b8529166e22a5013079075b0e1eab4d4c838e527)）。

更新到 **2.2.8**（2018-05-27）后就可以正常使用这个设置。

但是对数据库的缓存更新还是没作用，原因就是还有个报表定义的缓存没有清除

## 禁用报表定义缓存

参考 `com.bstek.ureport.cache.CacheUtils#setApplicationContext` 内的写法，可以发现是有个 ReportDefinitionCache 接口实现，并默认拿第一个做报表定义缓存，如果一个实现都没有的话，会生成下面的 DefaultMemoryReportDefinitionCache 来做缓存。

遂复写 DefaultMemoryReportDefinitionCache，加上一个可以清除的接口，通过 post 请求 /clearCache 调用就行了

### [ErasableMemoryCache](https://github.com/yingw/ureport-demo/blob/master/src/main/java/cn/wilmar/ureport/report/cache/ErasableMemoryCache.java)
```java
/**
 * 类似 DefaultMemoryReportDefinitionCache，增加 clearCache 方法来清除缓存，用于在后台改了报表没法立即生效的场景。
 * 至于初始化这个和 DefaultMemoryReportDefinitionCache 的关系，参考 com.bstek.ureport.cache.CacheUtils#setApplicationContext
 * @author Yin Guo Wei 2018/5/27.
 */
@Component
public class ErasableMemoryCache implements ReportDefinitionCache {

    private Map<String, ReportDefinition> reportMap = new ConcurrentHashMap<String, ReportDefinition>();

    /**
     * 清除缓存
     */
    public void clearCache() {
        this.reportMap.clear();
    }

    @Override
    public ReportDefinition getReportDefinition(String file) {
        return reportMap.get(file);
    }

    @Override
    public void cacheReportDefinition(String file, ReportDefinition reportDefinition) {
        if (reportMap.containsKey(file)) {
            reportMap.remove(file);
        }
        reportMap.put(file, reportDefinition);
    }
}

```

### CacheController
web 层入口

```java
/**
 * 接收前端发来 post 指令清除缓存
 * @author Yin Guo Wei 2018/5/27.
 */
@Controller
public class CacheController {

    private final ErasableMemoryCache erasableMemoryCache;

    public CacheController(ErasableMemoryCache erasableMemoryCache) {
        this.erasableMemoryCache = erasableMemoryCache;
    }

    /**
     * 清除缓存的入口
     * @return 返回首页
     */
    @PostMapping("/clearCache")
    public String clearCache() {
        erasableMemoryCache.clearCache();
        return "redirect:/reports";
    }
}
```

### NullCache
同时如果想完全不用缓存，还定义了一个 NullCache，但是不建议使用，只适合于开发环境
```java
/**
 * 完全不适用 Cache，建议不开启。如果需要开启，加上类上注解 @Component。
 * （建议使用 ErasableMemoryCache）
 * @author Yin Guo Wei 2018/5/27.
 */
//@Component
public class NullCache implements ReportDefinitionCache {
    @Override
    public ReportDefinition getReportDefinition(String file) {
        System.out.println("NullCache.getReportDefinition");
        System.out.println("file = " + file);
        return null;
    }

    @Override
    public void cacheReportDefinition(String file, ReportDefinition reportDefinition) {
        System.out.println("NullCache.cacheReportDefinition");
    }
}
```

注意 NullCache 和 ErasableMemoryCache 只能定义一个 Component，原因是 Java 反射返回的实现没有排序，UReport 只取了 getBeansOfType(ReportDefinitionCache.class) 的第一个。（当然也可以用 @Order 来排序）

## 测试

用 postman 提交 post 请求：http://localhost:8080/clearCache，观察多次打开后台的 log 以及断点，测试通过，开关生效。

最后：不高兴在 Reports 页面放一个提交按钮到 clearCache 请求。

## 其他类参考

一些断点的地方
- com.bstek.ureport.cache.CacheUtils#getReportDefinition
- com.bstek.ureport.console.designer.DesignerServletAction#loadReport
- com.bstek.ureport.console.designer.DesignerServletAction#deleteReportFile
- com.bstek.ureport.console.cache.HttpSessionReportCache
- com.bstek.ureport.cache.DefaultMemoryReportDefinitionCache