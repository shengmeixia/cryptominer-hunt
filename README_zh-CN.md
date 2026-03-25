# cryptominer-hunt

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)
[![Platform](https://img.shields.io/badge/Platform-Linux-lightgrey)](https://github.com)
[![Security](https://img.shields.io/badge/Security-Incident%20Response-red)](https://github.com)

[English](README.md) | **简体中文** | [日本語](README_ja.md)

一款基于 AI 的技能工具，用于检测、调查和清除 Linux 服务器上的加密货币挖矿木马及相关恶意软件。

## 为什么需要它

加密货币挖矿木马是被入侵 Linux 服务器上最常见的恶意载荷之一。要彻底清理它，需要追踪完整的攻击链 —— 矿机、投放器、持久化机制、入口点和横向移动 —— 涉及数十个可能的隐藏位置。这项工作繁琐、容易遗漏，而且很难做到彻底。

**cryptominer-hunt** 将 AI 编程助手转变为应急响应工具。它不采用死板的检查清单，而是定义*目标和原则*，让 AI 根据每台服务器的实际环境自适应地调整响应策略。它像人类安全分析师一样进行调查：追踪证据、对抗性思维、追查每一条线索直到穷尽。

## 功能概览

| 阶段 | 目标 |
|------|------|
| **初步分类** | 发现 CPU 异常、可疑进程、异常网络活动和不应存在的文件 |
| **深度调查** | 对每个指标：追踪二进制文件、映射持久化机制、检查横向移动、验证系统完整性、定位入口点 |
| **报告** | 结构化报告 —— 恶意软件清单、持久化机制、网络/文件 IOC、重建攻击链、风险评估 |
| **根除** | 终止进程（先杀看门狗）、删除文件、清理持久化、封锁 C2 基础设施 |
| **验证** | 从各个维度确认系统已清理干净 |
| **建议** | 凭证轮换、补丁修复、横向审计目标、监控措施、是否需要重装评估 |

## 安装

将技能文件复制到 AI 编程助手的技能目录中：

```bash
# 创建技能目录
mkdir -p ~/.claude/skills/cryptominer-hunt

# 复制技能文件
cp SKILL.md ~/.claude/skills/cryptominer-hunt/SKILL.md
```

或直接克隆本仓库：

```bash
git clone https://github.com/shengmeixia/cryptominer-hunt.git ~/.claude/skills/cryptominer-hunt
```

## 使用方法

在 Linux 服务器的终端会话中：

```
# 完整扫描 —— 调查并清理（执行破坏性操作前会请求确认）
/cryptominer-hunt

# 试运行 —— 仅调查和报告，不做任何修改
/cryptominer-hunt --dry-run
```

### 使用场景

- 服务器出现无法解释的高 CPU 占用
- 怀疑服务器被入侵或已确认被入侵
- 事件处理后，验证清理是否彻底
- 对面向互联网的服务器进行常规安全审计

## 设计理念

本技能刻意采用**目标导向而非指令式**的设计。它定义*要调查什么*，而不是*怎么调查*。

- **零硬编码命令。** AI 根据所在操作系统和环境自行选择合适的工具。
- **证据驱动。** 每项发现都会触发更深入的调查 —— 发现矿机意味着存在投放器，投放器意味着存在入口点，入口点意味着存在持久化机制。
- **对抗性思维。** 技能引导 AI 考虑各种规避技术、隐藏进程、被篡改的系统二进制文件，以及它可能忽略的攻击模式。
- **并行执行。** 独立的检查项同时运行，加快初步分类速度。

这意味着该技能无需修改即可在 Debian、Ubuntu、RHEL、Alpine、Arch 及任何其他 Linux 发行版上运行。

## 系统要求

- 支持自定义技能的 AI 编程助手
- Linux 服务器（任意发行版）
- 目标服务器的 root 或 sudo 权限

## 实战验证

本技能是在对一台遭受入侵的生产服务器进行实际应急响应过程中开发和完善的。该服务器通过 Next.js RCE 漏洞被攻破。调查发现：

- 多个 XMRig 矿机部署，目标为门罗币矿池
- IoT 僵尸网络二进制文件（Mirai/Gafgyt 变种）
- 三台轮换的 C2 服务器通过 netcat 投递恶意载荷
- 通过 shell 配置文件注入和 cron 任务实现持久化
- UPX 加壳的 ELF 二进制文件部署在 /tmp、/dev/shm 和应用目录中

该技能的设计在事件处理过程中经历了三次迭代，从一个固定的命令清单演变为你现在看到的自适应、目标导向的框架。

## 许可证

[MIT](LICENSE)

## 贡献

欢迎贡献！如果你在事件响应中使用过该技能并发现了它遗漏的攻击模式或隐藏位置，请提交 issue 或 PR。
