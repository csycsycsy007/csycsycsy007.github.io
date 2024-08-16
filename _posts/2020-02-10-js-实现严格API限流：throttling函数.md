---
title: 实现严格 API 限流： throttling 函数
date: 2020-02-10 08:00:00 +0800
categories: [js]
tags: [node]
---

## 背景

在现代 Web 应用中，用户频繁的重复请求可能会对系统资源造成巨大压力，甚至引发潜在的安全问题。前端的防抖或节流机制尽管能在一定程度上减少重复请求，但无法从根本上杜绝恶意行为。为了更好地保护后端服务的稳定性，我们通常会在服务端实现请求限流。本文将介绍一个基于 Node.js 和 Redis 的限流函数 `throttling`，并探讨其在严格 API 限流中的应用。

## 核心代码

下面是实现限流功能的 `throttling` 函数：

```javascript
/**
 * ip                  标识用户身份
 * throttlingPeriod    限流时间，单位 s
 * apiThrottlingLimit  限流时间内的最大请求次数
 */
throttling: async function throttling(ip, cacheKeySuffix, throttlingPeriod, apiThrottlingLimit) {
  try {
    if (!ip) {
      return { 'errCode': 401 };
    }

    throttlingPeriod = throttlingPeriod || 1;
    let expireType = 'EX';
    if (throttlingPeriod < 1) {
      expireType = 'PX';
      throttlingPeriod = throttlingPeriod * 1e3;
    }

    apiThrottlingLimit = parseInt(apiThrottlingLimit) || 1;

    const key = 'api.throttling.' + ip + '.' + cacheKeySuffix;

    const count = await redis.incrby(key, 1);

    await redis.set(key, count, expireType, parseInt(throttlingPeriod));

    if (count > apiThrottlingLimit) {
      return { 'errCode': 403 };
    }
  } catch (e) {
    return { 'errCode': 500 };
  }
},
```
