# 06 · 速查表与 FAQ

---

## 1. 一页速查

### 最小可用模板

```yaml
---
name: my-skill
description: <一句话能力>. This skill should be used when <具体场景 / 文件类型 / 任务>. Triggers: "<说法 1>", "<说法 2>".
---

# My Skill

## Overview
<1-2 句说明这个技能让 AI 能做成什么>

## When to use
<哪些请求该用；Do not use for ...>

## Steps
1. <祈使句步骤>
2. <祈使句步骤>

## Resources
- <正文中显式引用到的每个资源路径与用途>
```

> 注意：填写真实 `description` 时不要保留示例里的尖括号占位符，校验脚本会拒绝 `<` `>`。

### 目录结构

```text
skill-name/
├── SKILL.md        # 必需
├── scripts/        # 可选：执行
├── references/     # 可选：按需阅读
└── assets/         # 可选：用于产出
```

### frontmatter 字段

| 字段 | 必需 | 规则 |
| --- | --- | --- |
| `name` | 是 | `^[a-z0-9-]+$`；不以 `-` 首尾；不含 `--`；建议 ≤ 40 字符；与目录名一致 |
| `description` | 是 | 第三人称；做什么 + 何时用 + 触发说法；不含 `<` `>`；多行用 `\|` |
| `license` / `allowed-tools` / `disable` | 否 | 部分内置技能可见；支持度以当前环境为准 |

### 三层加载

| 层级 | 内容 | 时机 | 体量 |
| --- | --- | --- | --- |
| ① | `name` + `description` | 常驻 | 约 100 词 |
| ② | `SKILL.md` 正文 | 触发时 | < 5k 词 |
| ③ | 附带资源 | 按需 | 脚本可执行，理论不限 |

### 资源判断

| 需求 | 放哪 |
| --- | --- |
| 反复重写的同一段代码 / 要确定性 | `scripts/` |
| 长规范、schema、API 文档、详细流程 | `references/` |
| 模板、样板、图片、字体、示例数据 | `assets/` |
| 核心流程几句话 | `SKILL.md` 正文 |

### 常用命令

```bash
SKILL_CREATOR="/Applications/CodeBuddy CN.app/Contents/Resources/app/extensions/genie/out/extension/builtin/skill-creator"

# 初始化（--path 务必显式传入）
python3 "$SKILL_CREATOR/scripts/init_skill.py" my-skill --path ~/.codebuddy/skills

# 校验
python3 "$SKILL_CREATOR/scripts/quick_validate.py" ~/.codebuddy/skills/my-skill

# 校验并打包成 ./dist/my-skill.zip
python3 "$SKILL_CREATOR/scripts/package_skill.py" ~/.codebuddy/skills/my-skill ./dist
```

### 校验规则（内置 `quick_validate.py`）

| # | 检查 |
| --- | --- |
| 1 | `SKILL.md` 存在 |
| 2 | 文件以 `---` 开头且 frontmatter 正确闭合 |
| 3 | 含 `name:` 与 `description:` |
| 4 | `name` 为合法 hyphen-case |
| 5 | `description` 不含 `<` / `>` |

### 存放位置

| 范围 | 路径 |
| --- | --- |
| 用户级（所有项目） | `~/.codebuddy/skills/<name>/` |
| 项目级（随仓库共享） | `<项目>/.codebuddy/skills/<name>/` |

---

## 2. 常见错误对照表

| 现象 | 原因 | 修复 |
| --- | --- | --- |
| 技能从不生效 | 放在普通目录 / 没有 `SKILL.md` / 描述太泛 | 移到 skills 目录；补全描述中的触发场景 |
| 到处误触发 | 描述边界过宽 | 加 `Do not use for …`，收窄到具体文件类型 / 任务 |
| 触发了别的技能 | 多个描述重叠 | 各自写清负责范围 |
| 校验报 `should be hyphen-case` | `name` 用了大写 / 下划线 / 中文 / 连续连字符 | 改为小写连字符 |
| 校验报 `angle brackets` | `description` 含 `<` `>`，或用了 `description: >` | 改单行或 `description: \|` |
| 校验报 `No YAML frontmatter found` | 文件不是以 `---` 首行开头，或闭合缺失 | 补齐首尾 `---` |
| 初始化直接打印用法退出 | 省略了 `--path` | 显式传 `--path` |
| 打包没有生成 zip | 校验未通过 | 按报错修好后重试 |
| 正文太长、结果跑偏 | 把长资料塞进正文 | 移入 `references/`，正文留引用 |
| 脚本 / 模板从未被使用 | 正文没引用资源 | 写明路径与调用时机 |
| 每次输出格式不同 | 没有固定模板 | 用 `assets/` 模板 + 明确输出格式 |
| 换机器失效 | 写死本机绝对路径 / 依赖未说明 | 用相对路径，注明依赖 |

---

## 3. FAQ

**Q1：Skill 和 Rules 到底怎么选？**
每次都必须遵守 → Rules；特定任务才需要的一整套流程与资料 → Skill。两者可同时存在，Skill 正文不要重复 Rules 内容。

**Q2：把 `SKILL.md` 放在项目任意目录能生效吗？**
不能。需放在 `~/.codebuddy/skills/<name>/` 或 `<项目>/.codebuddy/skills/<name>/`，且文件名必须是 `SKILL.md`。

**Q3：一定要打包成 zip 吗？**
本机使用不必打包；分发给同事 / 其他机器时才打包。打包前会自动跑校验。

**Q4：正文能用中文写吗？**
可以。官方要求的是「祈使句 / 指令化、不用第二人称」，不是语言限制；`description` 里建议中英文触发说法都写，便于命中。

**Q5：`SKILL.md` 能写多长？**
建议正文 < 5k 词。更长的资料放 `references/`，超过 1 万词时在正文给出 grep 搜索模式。

**Q6：怎么知道技能有没有被触发？**
用 `description` 中的原句直接提问；或在正文放一句独特的可观察指令（如输出固定标记）作探针。详见 [05-进阶功能.md](05-进阶功能.md) 第 7 节。

**Q7：Skill 能调用外部 API / 线上数据吗？**
Skill 本身只是指令。连接外部系统应由 MCP / 工具完成，Skill 负责写清「何时调用、参数、失败如何回退」。

**Q8：多个技能职责重叠怎么办？**
在每个 `description` 与正文 `When to use` 中写清边界与排除项；必要时合并技能或按产品 / 文件类型拆分。

**Q9：怎么临时禁用某个技能？**
若当前环境支持 `disable: false` 之类的开关可使用；更稳妥的做法是把目录移出 skills 路径或改名（先确认当前版本行为）。

**Q10：更新技能后要重新打包 / 重启吗？**
本机直接改文件即可，无需打包；分发给别人的 zip 需要重新生成。改动后建议重新跑一次校验与真实请求回归。

**Q11：`description: >` 为什么校验失败？**
校验脚本禁止 description 中出现 `<` 或 `>`，`>` 会被判为非法字符。改用 `description: |` 或单行。

**Q12：初始化生成了一堆示例文件，必须保留吗？**
不必。用不到的 `scripts/` `references/` `assets/` 示例文件与目录都删掉，并清空模板中的 `[TODO:]`。

**Q13：技能里能放密钥、内网地址吗？**
不要。项目级技能会随仓库共享；脚本也会被执行，敏感信息应通过环境变量 / 配置提供。

---

## 4. 术语表

| 术语 | 含义 |
| --- | --- |
| Skill | 以 `SKILL.md` 为核心的自包含能力包 |
| `SKILL.md` | 技能唯一必需文件，含 frontmatter 与 Markdown 指令 |
| Frontmatter | 文件顶部 `---` 之间的 YAML 元数据，至少含 `name` / `description` |
| 渐进式披露 | 元数据 → 正文 → 资源的三层按需加载机制 |
| Hyphen-case | 仅小写字母、数字、连字符的命名风格，如 `release-notes` |
| `scripts/` | 可直接执行、不必读入上下文的代码 |
| `references/` | 按需加载进上下文的参考资料 |
| `assets/` | 用于最终产出、不进入上下文的模板与素材 |
| Rules | 常驻生效的硬性约束（如 `CODEBUDDY.md`） |
| MCP | 连接外部系统与 API 的工具协议层 |
| Triggers | `description` 中列举的典型用户说法，用于提升触发命中率 |

---

[返回总览 README.md](README.md)
