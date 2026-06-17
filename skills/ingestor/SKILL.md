---
name: thought-ingestor
description: >
  批量摄入器。扫描主人放在固定目录（~/Desktop/日常记录/）中的
  语音文件和文字文件，逐个运行完整管线（STT→10Skill→归档），
  处理完毕移入"已处理"目录。
compatibility: pi
triggers: [处理日常记录, 摄入, 批量处理, ingest]
pipeline_position: 入口层 — 批量版 gatekeeper（调度 orchestrator 逐个处理）
---

# Thought Ingestor — 批量摄入

## 职责

当主人积累了离线录制的语音和文字文件后，一次性批量摄入思想体系。

## 目录结构

```
~/Desktop/日常记录/
├── 语音/          ← 放 .wav .mp3 .m4a .webm
├── 文字/          ← 放 .md .txt
└── 已处理/        ← 处理完的自动移到这里
```

主人随时可以把文件扔进"语音"或"文字"目录，然后在会话中说"处理日常记录"。

## 处理流程

```
扫描 语音/ 和 文字/ 目录
    │
    ▼ 对每个文件：
    │
    ├── .m4a/.wav/.mp3/.webm → STT（faster-whisper）→ 转录文本
    ├── .md/.txt              → 直接读取文本
    │
    ├── 文本 → thought-orchestrator（全管线）
    │     gatekeeper → distiller → prober → perspector
    │     → validator → mapper → linker → formatter
    │
    ├── 归档完成后 → mv 到 已处理/
    │
    └── 向主人汇报每条的处理结果
```

## 调用方式

### 被动模式（手动触发）

```
主人: "处理日常记录"
小皮:
  1. ls ~/Desktop/日常记录/语音/   → 3 个文件
  2. ls ~/Desktop/日常记录/文字/   → 1 个文件
  3. "共 4 个文件待处理，开始逐个处理..."
  4. 对每个文件：
     - STT（如需要）
     - 调用 orchestrator 全管线
     - 汇报："✔ voice_001.m4a → 02_方法论/洞察力.md"
     - mv 到已处理/
  5. "全部处理完成。4 个文件已归档，文件已移入已处理/。"
```

### 主动模式（定时任务，后续实现）

通过 pi-daemon 添加定时任务：
```
*/30 * * * * 扫描日常记录目录 → 有新文件 → 自动处理
```

## 注意事项

1. **批量处理的上下文**：每个文件独立走管线，不混淆
2. **失败处理**：某个文件处理失败不影响其他文件，失败文件留在原目录并标记
3. **去重**：文件名 + MD5 校验，已处理过的文件自动跳过
4. **大文件**：语音 > 10 分钟的，提醒主人分段
5. **空文件**：小于 100 字节的文字文件视为空，跳过

## 与其他 Skill 的关系

```
thought-ingestor（批量入口）
    │
    ├─→ faster-whisper（STT，复用 system-tool 的 stt.py）
    │
    └─→ thought-orchestrator（全管线编排）
         ├─→ gatekeeper
         ├─→ distiller
         ├─→ prober
         ├─→ perspector
         ├─→ validator → casebook
         ├─→ mapper
         ├─→ linker
         └─→ formatter
```
