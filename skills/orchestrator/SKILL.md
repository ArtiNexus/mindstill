---
name: thought-orchestrator
description: >
  思想管线总编排。按序调度 gatekeeper → distiller → prober → perspector
  → validator → mapper → linker → formatter。
  不暂停等确认。遇到问题标记继续。最后汇报。
compatibility: pi
triggers: [记一下, 存档, 归档, 存一下, 记下来, 帮我存,
          我跟你说, 你听一下, 你看看这个, 我突然发现,
          我想到了, 我有个想法, 等会儿说, 我分析,
          我的判断是, 很有可能是, 往往就是]
pipeline_position: 总控
---

# Thought Orchestrator — 总编排

## 职责

按顺序跑完所有子 Skill。自动。不停。遇到问题标记，继续走。

## 流程图

```
主人触发词
    │
    ├─ 明确归档: "记一下""存档"等 → 全自动到底
    └─ 暗示表达: "我跟你说""我突然发现"等 → gatekeeper 先判
                                                    ├─ keep → 继续
                                                    └─ discard → 不存档
    │
    ▼
[gatekeeper]  有判断?有客观锚点?
    │ keep
    ▼
[distiller]   识别结构→找结论标记词→蒸馏
    │
    ▼
[prober]      三层追问→够用了就停
    │
    ▼
[perspector]  五维透视→维度不够就标注
    │
    ▼
[validator]   对比已有体系→匹配/冲突/新
    │
    ▼
[mapper]      多标签映射
    │
    ▼
[linker]      维度相交检测→建立关联
    │
    ▼
[formatter]   时间+描述+结论及依据→写入
    │
    ▼
 📋 汇报: 存了什么、放哪了、有冲突的提醒
    │
    ▼
[casebook]    归档后追加案例记录（如果是完成态+有代表性）
```

## 核心规则

1. **全自动**：不是 Web 界面，不需要暂停等确认
2. **不卡住**：某步失败 → 标记 → 继续
3. **冲突是生长点**：validator 发现冲突 → 照样归档 + 标记"矛盾待解"
4. **相关性验证**：全部 Skill 完成后检查 Skill 之间的数据传递是否闭合

## 降级

| 故障 | 处理 |
|:-----|:-----|
| gatekeeper 返回 discard | 终止，不存档 |
| distiller confidence < 0.3 | 用原文进 perspector，跳过 prober |
| 案例库为空 | validator 标记"冷启动"，继续 |
| linker 无文件可读 | 跳过关联，标记 |
| formatter 写入失败 | 保存到 `05_灵感/未归档.md`，通知主人 |

## 汇报模板

```
已归档:
  → 02_方法论与思维链/洞察力.md   "阳光通过体感激活情绪"
  → 00_认知地基/物质与意识.md     (引用指针)

标记:
  ⚡ 1条与核心价值观有张力: "自洽就是真理？" → 矛盾待解
```

## 演化记录

| 日期 | 版本 | 变更 |
|:-----|:-----|:-----|
| 2026-06-17 | v1.0 | 创建框架 |
| 2026-06-17 | v1.1 | 去掉暂停等确认、明确全自动、汇报模板 |
