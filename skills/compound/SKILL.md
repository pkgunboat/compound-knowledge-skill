---
name: compound
description: 从当前对话中提取已解决的问题或学到的知识，生成结构化文档存入全局知识库 ~/.claude/knowledge/solutions/。在解决 bug、完成调试、掌握新模式后使用。当对话中出现"搞定了"、"问题解决了"、"原来是这样"等信号时也可触发。
---

# /compound — 全局知识沉淀

## 目标

从当前对话中提取有价值的经验，生成结构化文档，存入全局知识库供未来跨项目复用。

## 存储位置

```
~/.claude/knowledge/solutions/[category]/[filename].md
```

## 执行流程

### Phase 1: 分析对话上下文

回顾当前对话，识别：
- 解决了什么问题？或学到了什么？
- 根因是什么？
- 尝试过哪些无效方案？
- 最终方案是什么？为什么有效？
- 未来如何预防？

### Phase 2: 分类与查重

**自动分类到以下目录之一：**

| 目录 | 适用场景 |
|------|---------|
| `build-errors/` | 构建、编译、依赖错误 |
| `test-failures/` | 测试失败、断言错误 |
| `runtime-errors/` | 运行时异常、崩溃 |
| `performance-issues/` | 性能瓶颈、内存泄漏 |
| `database-issues/` | 数据库相关问题 |
| `security-issues/` | 安全漏洞、权限问题 |
| `ui-bugs/` | 前端/界面问题 |
| `integration-issues/` | API、第三方服务集成问题 |
| `logic-errors/` | 业务逻辑缺陷 |
| `config-issues/` | 配置、环境相关问题 |
| `patterns/` | 通用模式、最佳实践、工作流经验 |
| `tools/` | 工具使用技巧、CLI 命令、开发环境 |

**查重：** 用 Grep 搜索 `~/.claude/knowledge/solutions/` 中是否有高度相似的文档。

| 重叠程度 | 判断标准 | 处理方式 |
|---------|---------|---------|
| 高 | 同一问题、同一根因、同一方案 | 更新已有文档，添加 `last_updated` |
| 中 | 相关但不同的变体 | 新建文档，添加 `related` 字段交叉引用 |
| 低/无 | 全新问题 | 正常新建文档 |

### Phase 3: 生成文档

根据内容性质选择对应 track：

#### Bug Track（问题修复类）

```markdown
---
title: [简明标题]
category: [分类目录名]
track: bug
tags: [tag1, tag2, tag3]
project: [项目名称，可选]
created: [YYYY-MM-DD]
last_updated: [YYYY-MM-DD]
related: [相关文档路径列表，可选]
---

## 问题描述
[具体的错误现象和触发条件]

## 症状表现
[用户/开发者可观察到的具体表现]

## 尝试过的无效方案
[列出走过的弯路，以及为什么无效——这往往是最有价值的部分]

## 解决方案
[最终有效的修复方案，包含关键代码片段]

## 根因分析
[为什么这个方案有效？底层机制是什么？]

## 预防措施
[未来如何避免同类问题]
```

#### Knowledge Track（知识经验类）

```markdown
---
title: [简明标题]
category: [分类目录名]
track: knowledge
tags: [tag1, tag2, tag3]
project: [项目名称，可选]
created: [YYYY-MM-DD]
last_updated: [YYYY-MM-DD]
related: [相关文档路径列表，可选]
---

## 背景
[什么场景下会用到这个知识]

## 核心要点
[具体的指导内容，包含代码示例]

## 为什么重要
[不了解这个知识会踩什么坑]

## 适用条件
[什么时候该用、什么时候不该用]

## 实际案例
[从当前对话中提取的真实例子]
```

### Phase 4: 写入文件

1. 确保目标分类目录存在（不存在则创建）
2. 文件名格式：`[简短描述-kebab-case].md`
3. 用 Write 工具写入文件

### Phase 5: 确认输出

输出摘要：

```
已沉淀知识:
  文件: ~/.claude/knowledge/solutions/[category]/[filename].md
  类型: [bug-track / knowledge-track]
  标签: [tags]
  摘要: [一句话总结]
```

## 关键原则

1. **只记录有复用价值的内容** — 不是所有 bug 都值得记录，只记那些根因不明显、容易反复踩坑的
2. **重视"无效方案"** — 走过的弯路往往比正确答案更有价值
3. **保持简洁** — 每份文档聚焦一个问题/知识点，不要塞太多内容
4. **标签要实用** — 用未来搜索时会想到的关键词作为标签
5. **项目无关性** — 文档应尽量抽象到可跨项目复用的程度；如果是项目特定的，在 `project` 字段标注
