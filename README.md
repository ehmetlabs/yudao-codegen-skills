# yudao-codegen-skills

Yudao / RuoYi-Vue-Pro 代码生成（codegen）技能定义仓库。  
本仓库用于维护 **触发契约、工作流、边界规则、评估用例**，帮助 Agent 在目标业务仓库中稳定执行单表 / 树表 / 主子表的模板化代码生成。

---

## 目录

- [1. 仓库定位](#1-仓库定位)
- [2. 你会在这里找到什么](#2-你会在这里找到什么)
- [3. 何时触发此技能](#3-何时触发此技能)
- [4. 不应触发的场景](#4-不应触发的场景)
- [5. 快速开始（维护者视角）](#5-快速开始维护者视角)
- [6. 标准工作流（模板驱动）](#6-标准工作流模板驱动)
- [7. 场景分支与模板类型](#7-场景分支与模板类型)
- [8. 前端类型映射（Front Type）](#8-前端类型映射front-type)
- [9. 边界规则（Boundary）](#9-边界规则boundary)
- [10. 评估体系（Evals）](#10-评估体系evals)
- [11. 命令与验证说明](#11-命令与验证说明)
- [12. 与外部目标仓库的关系](#12-与外部目标仓库的关系)
- [13. 文档更新规范](#13-文档更新规范)
- [14. FAQ](#14-faq)
- [15. 参考链接](#15-参考链接)

---

## 1. 仓库定位

这是一个 **文档/契约仓库**，不是可运行的服务仓库。

- 主要交付：技能定义与评估基线
- 主要用途：规范 Agent 对 Yudao codegen 的触发、执行和止损行为
- 非目标：在本仓库直接运行业务代码生成、构建、单测

> 简单说：这里维护“怎么做”和“何时停”，不是维护“业务系统本体”。

---

## 2. 你会在这里找到什么

当前核心文件：

- `SKILL.md`
  - 技能主规范（触发条件、工作流、边界、Do/Don't、参考锚点）
- `evals/rubric.md`
  - 评估量规（6 个维度、通过阈值、评分切线）
- `evals/benchmark-cases.md`
  - 基准用例（P/N/C/B 分组）
- `examples/scenario-matrix.md`
  - 场景矩阵（单表/树表/主子表关键线索）
- `AGENTS.md`
  - 本仓库内 agent 执行约束与编辑准则

推荐阅读顺序：

1. `SKILL.md`
2. `evals/rubric.md`
3. `evals/benchmark-cases.md`
4. `examples/scenario-matrix.md`
5. `AGENTS.md`

---

## 3. 何时触发此技能

当用户请求明显属于 Yudao / RuoYi-Vue-Pro codegen 工作流时触发，例如：

- 提到 `yudao` / `ruoyi-vue-pro` + `codegen` / `CRUD` / `scaffold`
- 明确要求：
  - 单表生成
  - 树表生成
  - 主子表生成
- 希望直接依据仓库 `codegen` 模板渲染输出文件

典型正向输入可参考：`evals/benchmark-cases.md` 中 `P-01` ~ `P-08`。

---

## 4. 不应触发的场景

以下请求不属于本技能主路径：

- 通用 CRUD（未限定 Yudao / RuoYi-Vue-Pro）
- 其他脚手架（如 JHipster、MyBatis-Plus Generator）
- 手写 Controller / Service 的普通开发任务
- 部署、运维、性能调优、权限配置等非 codegen 渲染任务

详见：`evals/benchmark-cases.md` 中 `N-01` ~ `N-08`。

---

## 5. 快速开始（维护者视角）

### Step 1：先确认需求是否属于本技能

- 具备 Yudao 语境 + codegen 语义，才进入主路径
- 否则按负向/澄清路径处理

### Step 2：按 Source of Truth 顺序定位规则

1. `SKILL.md`
2. `evals/rubric.md`
3. `evals/benchmark-cases.md`
4. `examples/scenario-matrix.md`

### Step 3：修改后做一致性复核

- 触发契约是否与 benchmark 一致
- 边界处理是否与 rubric 一致
- 场景矩阵是否仍可解释单表/树表/主子表差异

---

## 6. 标准工作流（模板驱动）

```text
识别仓库信号
  -> 读取 codegen 模板与映射
  -> 检查 yudao.codegen/front-type
  -> 执行前置检查（注释、结构、场景）
  -> 收集 schema 输入
  -> 推导默认值
  -> 选择模板分支
  -> 预览输出文件集合与路径映射
  -> 渲染模板
  -> 校验输出路径并落盘
```

关键点：

- 这是“模板驱动渲染”流程，不依赖 `/infra/codegen` API 调用链路
- 仓库识别采用 `Hard Signals`（必须命中）+ `Soft Signals`（仅辅助）的分层策略，软信号不能替代硬信号
- 前置检查失败时必须停止，不可“猜着继续生成”

---

## 7. 场景分支与模板类型

| 场景 | 模板类型 | 关键要求 |
|---|---|---|
| 单表 | `ONE(1)` | 普通 CRUD，无树结构、无主子表关系 |
| 树表 | `TREE(2)` | 需配置 `treeParentColumnId` 与 `treeNameColumnId` |
| 主子表（普通） | `MASTER_NORMAL(10)` + `SUB(15)` | 子表存在且 join 字段合法 |
| 主子表（ERP） | `MASTER_ERP(11)` + `SUB(15)` | 同上，面向 ERP 交互 |
| 主子表（内嵌） | `MASTER_INNER(12)` + `SUB(15)` | 同上，偏行内编辑 |

补充：

- `SUB(15)` 是主子表子表模板，不是独立主表场景
- 树表与主子表不得互判（rubric 明确要求）

---

## 8. 前端类型映射（Front Type）

| 值 | 枚举 | 说明 |
|---|---|---|
| 10 | `VUE2_ELEMENT_UI` | Vue2 遗留栈 |
| 20 | `VUE3_ELEMENT_PLUS` | 常见默认值 |
| 30 | `VUE3_VBEN2_ANTD_SCHEMA` | Vben2 + Antd Schema |
| 40 | `VUE3_VBEN5_ANTD_SCHEMA` | Vben5 + Antd Schema |
| 41 | `VUE3_VBEN5_ANTD_GENERAL` | Vben5 + Antd General |
| 50 | `VUE3_VBEN5_EP_SCHEMA` | Vben5 + Element Plus Schema |
| 51 | `VUE3_VBEN5_EP_GENERAL` | Vben5 + Element Plus General |
| 60 | `VUE3_ADMIN_UNIAPP_WOT` | Uniapp/WOT 跨端 |

判定顺序建议：

1. `application.yaml` 的 `yudao.codegen.front-type`（若为默认 `20`，仅可作为 fallback；需与模板输出路径和目标仓库结构一致）
2. `CodegenEngine.FRONT_TEMPLATES` 对应模板输出路径
3. 目标仓库目录结构（是否 cloud、多前端目录形态）
4. 若仍冲突/不确定：停止并向用户澄清

---

## 9. 边界规则（Boundary）

### 9.1 必须停止（Hard Stop）

- 表名不符合 `module_business` 约定，无法稳定推导 `moduleName` / `businessName`
- 表注释缺失
- 字段注释缺失
- 树表缺少 `treeParentColumnId` 或 `treeNameColumnId` 映射
- 主子表缺子表，或子表未绑定主表，或 `subJoinColumnId` 缺失/无效
- front-type 证据冲突且无法判定目标栈

### 9.2 可继续但必须显式提示

- 无主键：可继续，但提示“按首字段兜底为主键”

### 9.3 应拒绝重复生成

- 输入 schema 对既有目标无有效变化时，应提示“输入未变化”，避免重复生成

边界用例见：`evals/benchmark-cases.md` 的 `B-01` ~ `B-08`。

---

## 10. 评估体系（Evals）

`evals/rubric.md` 定义 6 个评审维度：

1. 触发准确性
2. 仓库识别
3. 场景分类
4. 工作流正确性
5. 前端类型映射
6. 边界纪律

常用判断：

- 总评 Pass 建议线：`>= 80`
- 且“触发准确性 / 工作流正确性 / 前端类型映射”需单独通过

---

## 11. 命令与验证说明

本仓库当前不提供可运行的 build/lint/test/单测入口。

- Build: N/A
- Lint: N/A
- Test: N/A
- Single test: N/A

可安全使用的本地检查命令：

- `git status`
- `git diff`
- `git ls-files`
- 通过检索工具进行内容搜索/核对

> 如果任务需要真实编译、测试或单测执行，请切换到目标业务仓库后再识别实际命令。

---

## 12. 与外部目标仓库的关系

本仓库中的 `SKILL.md` 引用了外部目标仓库的关键源码锚点（示例）：

- `CodegenServiceImpl.java`
- `CodegenBuilder.java`
- `CodegenEngine.java`
- `CodegenTemplateTypeEnum.java`
- `CodegenFrontTypeEnum.java`

这些锚点用于“规则对齐”和“行为解释”，不是本仓库内可直接运行的实现文件。

---

## 13. 文档更新规范

当你修改 `SKILL.md` 时，建议按以下顺序同步：

1. 更新 `SKILL.md` 主规范
2. 对齐 `evals/rubric.md` 评分口径
3. 更新 `evals/benchmark-cases.md`（P/N/C/B）
4. 检查 `examples/scenario-matrix.md` 是否仍一致
5. 最后检查 `AGENTS.md` 与 README 是否出现矛盾

编辑原则：

- 不虚构命令、不虚构源码路径
- 枚举值与常量命名保持原样
- 规则变更必须能在 benchmark/rubric 中被验证

---

## 14. FAQ

### Q1：为什么这里没有 `mvn test` / `npm test`？

因为这是技能定义仓库，不是业务应用仓库。此处维护的是“规则与评估”，不是可执行工程。

### Q2：`front-type: 20` 是不是永远代表最终前端栈？

不是。它可以是默认值，但仍需要结合模板输出路径和目标仓库结构综合判断。

### Q3：遇到树表/主子表配置不完整怎么办？

按边界规则停止主路径，先补齐映射关系（tree 字段映射、子表与 join 字段）再继续。

### Q4：何时需要向用户澄清？

当信息不足以确定场景或目标栈时（例如未给表名、未给主子关联字段、front-type 冲突）。

### Q5：Cursor/Copilot 规则文件在哪里？

当前仓库未发现以下文件：

- `.cursorrules`
- `.cursor/rules/`
- `.github/copilot-instructions.md`

---

## 15. 参考链接

- 单表文档：https://doc.iocoder.cn/new-feature/
- 树表文档：https://doc.iocoder.cn/new-feature/tree/
- 主子表文档：https://doc.iocoder.cn/new-feature/master-sub/
- 单元测试文档：https://doc.iocoder.cn/unit-test/

---

如果你是首次接触本仓库，建议从 `SKILL.md` 开始，然后按本 README 的“快速开始”顺序阅读，最后用 `evals/benchmark-cases.md` 做一次完整自检。
