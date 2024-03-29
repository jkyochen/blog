---
title: "有状态服务改造到无状态服务注意点"
tags:
- Node.js
- Backend
date: 2023-02-15T22:07:33+09:00
---

之前在摩西科技工作时候的小反思，现在总结一下。

<!--more-->

## 为什么要做有状态服务

1. 为了快速处理高并发的问题，我们把有竞争关系的请求都转发到同一个服务上（根据用户 id 或者竞争资源的 id 来转发），同时利用 Node.js 的单进程模型，在内存中操作数据，并异步保存。因为操作内存不会有 IO 操作，所以请求的处理顺序就是请求的到来顺序。
2. 异步保存会存在数据丢失的可能性，不过我们的业务没有直接和钱相关的，所以暂时降低了可用性的考量。

## 为什么又要改造成无状态服务

1. 我们希望可以接入 k8s，让 k8s 来管理日益增多的微服务，并提供动态扩缩容能力。
2. k8s 也可以做有状态服务，但是会很复杂，所以我们决定改造成无状态服务。

## 改造注意点

1. 之前把有竞争关系的请求到转发到同一个服务上，现在需要通过加锁或者让 Redis lua 脚本处理竞争问题。
2. 为了解决在扩容之后对数据库的压力，通过 Redis 来缓存数据，一般我们使用 Cache Aside Pattern 来读取和更新数据，更新的时候先把数据保存到数据库，再把缓存失效。
3. 在内存中缓存包含其他服务的数据一定要小心。
