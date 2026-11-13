# Minimal Defensive Coding

> 让防御性代码与真实风险相称，而不是给每一行都套上 `try/catch`。

一个面向 Codex 的 Agent Skill，用来减少 AI 生成代码中的过度防御：层层异常捕获、重复校验、静默 fallback、无依据的重试、猜测式兼容层和“万一”抽象。

## 问题

AI 写代码时经常把“健壮”理解成“多写分支”：

- 每个内部函数都 `try/except`，日志打完再原样抛出
- 必需字段缺失时返回 `""` / `0` / `[]`，让调用方猜
- 调用方已经保证非空，仍在每个函数里 `if not x: return`
- 所有异常都重试，包括非幂等操作
- 为不存在的旧格式、旧调用方预留兼容分支
- 一个实现就上 interface / factory / plugin

结果：真正的错误被吞掉，调用栈变长，测试覆盖一堆不可达分支，维护者不知道哪条路径才是正常的。

## 核心原则

**边界严，内部松。**

- 在信任边界校验一次：用户输入、网络、文件、环境变量、第三方数据、安全/数据完整性相关操作。
- 解析成内部可信类型后，内部代码信任契约，不重复检查。
- 只捕获当前层能处理的异常；不能处理就让它向上抛。
- 必需项 fail fast；默认值只用于明确可选的语义。
- 只对已知瞬态、幂等、有上限的故障重试。
- 出现第二个真实用例再抽象。

添加防御前问四个问题：

1. 这个失败真的可能发生吗？证据是什么？
2. 当前层能处理它吗？
3. 类型、边界校验或调用方是否已经保证？
4. 删掉它会不会削弱安全、数据完整性或资源释放？

## 安装

### Codex（全局）

```bash
git clone https://github.com/dengsmj123-hash/minimal-defensive-coding.git
mkdir -p ~/.codex/skills
cp -r minimal-defensive-coding ~/.codex/skills/
```

Windows PowerShell：

```powershell
git clone https://github.com/dengsmj123-hash/minimal-defensive-coding.git
New-Item -ItemType Directory -Force "$env:USERPROFILE\.codex\skills" | Out-Null
Copy-Item -Recurse -Force .\minimal-defensive-coding "$env:USERPROFILE\.codex\skills\"
```

> 复制的是包含 `SKILL.md` 的整个目录。安装后新开 Codex 会话生效。

### 仅当前项目

把 `minimal-defensive-coding/` 复制到项目的 `.codex/skills/` 下。

## 使用

自动触发：描述匹配“实现/重构/审查代码，涉及错误处理、校验、fallback、retry”等场景。

显式调用：

```text
$minimal-defensive-coding
```

示例：

```text
用 minimal-defensive-coding 审查这个 diff，只指出过度防御，不要改无关代码。
```

```text
按最小充分防御实现这个配置读取函数：必需字段缺失时明确报错，不要返回空配置。
```

## 它会做什么 / 不会做什么

| 会 | 不会 |
| --- | --- |
| 把校验集中到边界 | 删除认证、授权、注入防护 |
| 让内部代码信任类型和契约 | 删除事务、幂等、资源清理 |
| 必需项明确失败 | 把所有异常都吞掉 |
| 只对已知瞬态故障重试 | 对非幂等操作盲目重试 |
| 删除不可达分支 | 删除真实失败路径的处理 |
| 审查时按行给出保留/删除建议 | 借机大范围重写 |

## 文件结构

```text
.
├── SKILL.md
├── agents/
│   └── openai.yaml
├── LICENSE
└── README.md
```

`SKILL.md` 是技能正文；`agents/openai.yaml` 是 Codex 的展示信息。

## 兼容性

本仓库遵循 Agent Skills 的 `SKILL.md` 结构，主要面向 Codex。其他支持 `SKILL.md` 的 Agent 运行时也可以按对应目录规则安装。

## English

Minimal Defensive Coding is a Codex skill that keeps error handling, validation, defaults, retries, and compatibility code proportional to real risk. It validates once at trust boundaries, trusts internal contracts, catches only errors the current layer can handle, fails fast for required inputs, and preserves security, data-integrity, and resource-cleanup defenses.

Install by copying the folder to `~/.codex/skills/` or `<project>/.codex/skills/`, then invoke `$minimal-defensive-coding`.

## License

MIT © 2026 dengsmj123-hash
