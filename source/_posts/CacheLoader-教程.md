---
title: CacheLoader 教程
date: 2020-02-04 09:07:09
categories: BackEnd
tags:
    - Java
    - Cache
top:
---
# 1. LoadingCache

LoadingCache是指用CacheLoader建立的缓存。想要使用LoadingCache的话，就是使用方法`get(K)`。如果不在LoadingCache里面的话，就会使用CacheLoader做一次加载；如果有的话就直接拿回结果了。

# 2. 使用CacheLoader以及LoadingCache

    LoadingCache<String, String> loadingCache = CacheBuilder.newBuilder()
        .build(new CacheLoader<String, String>() {
            @Override
            public String load(final String s) throws Exception {
                return response;
            }
        });


+ CacheLoader在使用newBuilder()创造新实例的时候还可以做很多的设置，比如
    +  `expireAfterAccess(long duration, TimeUnit unit)`
        + 在每个entry被创建以后，经过一段固定的时间做自动移除操作
    + `maximumSize(long size)`
        + 定义cache能有的最大entry数量 

# Reference 

1. https://www.baeldung.com/guava-cacheloader 
2. https://stackoverflow.com/questions/43993731/what-is-a-loadingcache
3. https://guava.dev/releases/21.0/api/docs/com/google/common/cache/LoadingCache.html