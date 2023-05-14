---
title: Dev 碎碎念
categories: [dev stories]
tags: [journal]
---

# Tornado 试用思考

Tornado 在物联网上结合 twisted 蛮好用的。tornado 主要用于 websocket/tcp 这些长连接数据流的场景，结合 async 很有效，python 生态里暂时应该没有更好的替代框架。tornado支持把底层事件循环换成 asyncio 的循环。接着 gevent 还可以把 asyncio 的循环再 patch掉。这样 tornado 可以轻松异步调用任何支持 asyncio 或 gevent 的库。同时在 Windows 上使用 gevent (IOCP) 的 tornado 会有非常恐怖的并发能力。如果供 web CRUD 或者 restful 接口设计，优势不大。几个主流 python 框架更多是互补关系，不是竞争关系。
