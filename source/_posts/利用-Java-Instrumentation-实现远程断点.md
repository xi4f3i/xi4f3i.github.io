---
title: 利用 Java Instrumentation 实现远程断点
date: 2020-05-19 13:13:32
tags: [Java]
---

最近在线上问题排障时，发现了公司内部框架实现的一个非常有用的功能，可以对生产机器上的代码动态添加断点，不阻塞代码的运行，但是能够获取断点处的栈帧。

翻看源码了解了下实现思路。

# 基本原理

利用 Java Instrumentation 实现对类定义进行动态改变和操作。

# 加载 Agent

1. 实现 Agent 启动方法 `premain` 或 `agentmain`

2. Agent 打成一个 jar 包，并在 MANIFEST.MF 中指定

3. JVM 启动参数添加 `"-javaagent:xxx.jar"` 加载 Agent

# 断点注册

断点平台与 GitLab 中应用对应的代码库关联，打断点时收集所在的代码行和文件路径。

Agent 内实现一个 http 接口，接受断点平台发送的断点信息。

调用 Instrumentation.retransformClasses 将断点所在的类重新加载。

实现 ClassFileTransformer 的 transform 方法，在断点处添加记录栈帧的代码。

通过 Instrumentation.addTransformer 添加到实现的 ClassFileTransformer

# 获取栈帧信息

通过 HTTP 将断点处的栈帧数据发送给断点平台。
