# AI 编码规范（ai-coding-standards）

一套从真实业务项目中沉淀、面向 **AI 编程工具** 的编码规范：让 AI 写代码时有明确的红线可查，把返工挡在生成之前。

与流程类 skill 集 [ai-dev-workflow](https://github.com/gaoqiongxie/ai-dev-workflow)（管"怎么干活"）互补，本仓库管 **"代码长什么样算合格"**——命名、接口、事务、数据库、日志、安全、Git 纪律。

## 为什么专门做"AI 版"规范

AI 生成代码有一些人类少犯、AI 高频踩的坑，每条规则尽量标注 **"AI 为什么爱犯"**：

- 喜欢在组件里直接写 axios，绕过统一拦截器
- 遇到 push 被拒第一反应 `push -f`
- 把 `@Transactional` 标在私有方法上，或事务里调远程接口
- 生成 DDL 默认 `DEFAULT NULL`，埋 NPE 和三值逻辑
- 查全量实体改两个字段再整体 update，覆盖并发修改
- 在 for 循环里逐条查库
- 新增枚举值不扫全链路 `equals`/`switch`，上线即漏兼容

## 规则总览

| 文件 | 覆盖范围 | 适用 |
|------|---------|------|
| [rules/backend-java.md](rules/backend-java.md) | 命名 / 代码风格 / 接口设计（含外部调用 Proxy 层）/ 异常 / 事务 / 数据库（DDL+SQL+高危清单）/ Maven / 日志 / 安全 / i18n 内核 / 审查清单 | Java + Spring + MyBatis 系 |
| [rules/frontend-baseline.md](rules/frontend-baseline.md) | API 调用铁律 / 组件规范 / 模板规范 / 安全基线 / 错误处理 / 工程提交 | Vue 2/3（原则适用 React 等） |
| [rules/git-safety.md](rules/git-safety.md) | 操作权限矩阵 / 历史红线（禁 force push 等）/ 提交规范 / 提交前 Checklist | 所有把 Git 交给 AI 的场景 |
| [rules/change-discipline.md](rules/change-discipline.md) | 需求变更防返工五阶段：冻结前确认 → 设计强制章节 → 编码前影响扫描 → 自测场景 → 返工红线 | 需求开发全周期 |

## 最快接入：让 AI 把规范装进你的项目

把下面这段 prompt **原样**发给你项目里的 AI 编码助手（Claude Code / Trae / Kimi / Cursor 均可），它会把本仓库的规则合并进你项目的规则入口：

```text
我要给当前项目接入一套 AI 编码规范，仓库地址：
https://github.com/gaoqiongxie/ai-coding-standards

请先通读该仓库的 README.md 和 rules/ 下四个文件，然后帮我完成接入：

1. 先盘点我项目现状（语言、框架、现有规则入口文件），再从四个规则文件里挑适用的部分：
   - Java + Spring + MyBatis 后端 → backend-java.md
   - Vue 前端 → frontend-baseline.md
   - 任何把 Git 交给 AI 的场景 → git-safety.md（必装）
   - 需求开发全周期 → change-discipline.md
2. 把选定规则合并进我项目的规则入口（Claude Code 用 CLAUDE.md，Codex/通用 用
   AGENTS.md，Trae/Kimi 用 .trae/rules/），合并前先 diff 现有内容，冲突处列出对比
   给我看，确认后再落盘
3. 模板类占位（包名前缀、表名前缀、分支名、提交格式）按我项目实际约定替换，
   但每条规则「AI 为什么爱犯」的注释必须保留
4. 在规则入口里加一句总纲：以上规则仅对新增/修改代码生效，历史代码保持不动，不批量迁移

【红线】
- 只改规则入口文件，不因为我项目历史代码不符合规范就提出"批量重构"
- 不删改我项目已有的团队约定；与本规范冲突时列出来让我裁决，不要自行覆盖
- 接入完成后给我一份摘要：装了哪几份规则、替换了哪些占位、有哪些地方需要我人工裁决
```

## 怎么用

**方式一：项目规则文件（推荐）**

把对应文件内容合并进你项目的规则入口，AI 每次会话启动即读到：

- Claude Code：`CLAUDE.md`
- Codex / 通用：`AGENTS.md`
- Trae / Kimi：`.trae/rules/*.md`

**方式二：按需摘取**

只取你团队缺的部分（比如先上 git-safety），贴进任意 AI 工具的项目上下文。

**方式三：与 ai-dev-workflow 联动**

- `change-discipline.md` 阶段 1-2 ↔ `openspec-propose` 的需求探询与设计文档
- 审查清单 ↔ `low-rework-dev-gate` 防返工门禁
- 返工计数与结论 ↔ `daily-work-review` 日报沉淀

## 使用原则

- **规则仅对新开发代码生效**，历史代码保持不动，不批量迁移
- 所有规范为**模板基准**：包名、路径前缀、分支名、提交格式等按你项目实际调整，但"为什么"的部分建议保留
- 每条规则标注了 AI 高频踩坑点， review 时也可作为人类审查清单

## 脱敏说明

本仓库已从原始企业项目中完全脱敏：不含任何公司系统、内网地址、人名与联系方式。接口前缀、表名前缀、分支模型均为通用模板，接入时替换为你的团队约定即可。

## 相关项目

- [ai-dev-workflow](https://github.com/gaoqiongxie/ai-dev-workflow)：OpenSpec SDD 工作流 + 防返工门禁 + 共享记忆库（12 个 skill）
- [lark-claude-bot](https://github.com/gaoqiongxie/lark-claude-bot)：把这些规则喂给飞书 AI 开发助手的落地示例

## 许可

可自由用于学习与内部使用；欢迎提 Issue 交流你团队踩过的坑——好的规则都是从返工里长出来的。
