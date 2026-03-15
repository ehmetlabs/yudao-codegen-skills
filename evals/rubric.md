# Yudao-Codegen-Skills 评估量规

本文档定义 skill 质量的评审维度与评分标准，供人工复核或自动化评估使用。

---

## 评审维度

### 1. 触发准确性 (Triggering Accuracy)

| 等级 | 标准 | 信号词覆盖 |
|------|------|-----------|
| 优秀 | 正向用例 100% 触发，负向用例 0% 误触发 | yudao / ruoyi / codegen / CRUD / scaffold / 树表 / 主子表 |
| 合格 | 正向用例 >= 90% 触发，负向用例误触发 < 10% | 核心词命中即可 |
| 不合格 | 正向用例触发 < 90%，或负向用例误触发 >= 10% | 需重写触发契约 |

- 通过阈值：达到“合格”及以上；即正向用例触发率 `>= 90%`，且负向用例误触发率 `< 10%`。

### 2. 仓库识别 (Repository Detection)

验证 skill 能否正确识别目标仓库类型：

- [ ] 检测到 `yudao-module-infra` 模块存在
- [ ] 检测到 `src/main/resources/codegen/` 模板目录及模板族（java/sql/vue/vue3/vben等）
- [ ] 检测到 `CodegenEngine.java` 与 `CodegenBuilder.java` 核心引擎类
- [ ] 检测到 `application.yaml` 中的 `yudao.codegen.*` 配置
- [ ] 正确区分本地目标仓库与 ruoyi-vue-pro 家族变体

- 通过阈值：以上 5 项需 `5/5` 命中；其中前 4 项任一漏检直接不通过。
- 评分切线：`100 = 5/5`；`60 = 仅第 5 项不稳定`；`0 = <= 3/5` 或遗漏任一关键硬信号。

### 3. 场景分类 (Scenario Classification)

验证场景分支判断的准确性：

| 场景 | 判断依据 | 预期处理 |
|------|---------|---------|
| 单表 | 缺少父/子/树字段映射，按普通表处理 | 模板类型 = ONE |
| 树表 | 基于可配置的 `treeParentColumnId`（树父字段编号）与 `treeNameColumnId`（树名称字段编号）映射确定父节点和显示名称字段，非固定字段名 | 模板类型 = TREE |
| 主子表 | 识别 `MASTER_*` 主表模板与 `SUB` 子表模板的组合，并确认子表通过 `subJoinColumnId` 等配置建立与主表的关联 | 模板类型 = MASTER_SUB |

- 通过阈值：上表 3 类场景必须全部判对 `3/3` 才可通过，任一场景缺失或误判均判定为不通过；树表与主子表不得互判。
- 评分切线：`100 = 3/3`；`0 = <= 2/3` 或出现树表/主子表互判。

### 4. 工作流正确性 (Workflow Correctness)

标准流程检查点：

1. **前置检查** - 验证表结构注释完整性
2. **读取模板阶段** - 识别 `codegen` 模板目录与 `SERVER_TEMPLATES` / `FRONT_TEMPLATES`
3. **收集 schema 阶段** - 获取表结构、字段注释、主键、关联字段等输入
4. **配置阶段** - 确定模板类型、前端类型与命名默认值
5. **预览阶段** - 先确认将要输出的文件集合与路径映射
6. **生成阶段** - 直接渲染模板并写入目标模块
7. **后处理** - 最小化整理（校验输出路径、避免错写 repo 结构）

- 通过阈值：7 个检查点至少满足 `6/7`，且必须包含"前置检查、读取模板阶段、收集 schema 阶段、配置阶段、生成阶段"这 5 个主路径步骤。
- 评分切线：`100 = 7/7`；`85 = 6/7` 且主路径完整；`0 = <= 5/7` 或缺少任一主路径步骤。

### 5. 前端类型映射 (Front-Type Mapping)

验证 `CodegenFrontTypeEnum` 的正确使用：

| 类型值 | 名称 | 适用场景 |
|--------|------|---------|
| 10 | VUE2_ELEMENT_UI | 遗留 Vue2 项目 |
| 20 | VUE3_ELEMENT_PLUS | 默认推荐 |
| 30 | VUE3_VBEN2_ANTD_SCHEMA | Vben2 Ant Design Schema |
| 40 | VUE3_VBEN5_ANTD_SCHEMA | Vben5 Ant Design Schema |
| 41 | VUE3_VBEN5_ANTD_GENERAL | Vben5 Ant Design General |
| 50 | VUE3_VBEN5_EP_SCHEMA | Vben5 Element Plus Schema |
| 51 | VUE3_VBEN5_EP_GENERAL | Vben5 Element Plus General |
| 60 | VUE3_ADMIN_UNIAPP_WOT | Admin UniApp Wot 跨端 |

- 通过阈值：映射表中 8 个枚举值与名称需 `8/8` 完全正确，且评审结论不得引入表外枚举或改名。
- 评分切线：`100 = 8/8` 全对；`0 = 任一值、名称或对应关系错误`。

### 6. 边界纪律 (Boundary Discipline)

验证 skill 对以下边界条件的处理（边界范围以 SKILL.md、scenario-matrix.md 和 benchmark-cases.md 定义的包支持能力为准）：

| 边界类别 | 具体场景 | 预期处理 | 来源依据 |
|---------|---------|---------|---------|
| **命名规范** | 表名不符合 `module_business` 约定 | 拒绝生成，提示先规范表名再导入 | B-01 in benchmark-cases.md |
| **注释缺失** | 表注释为空 | 拒绝生成，提示先补充表注释 | B-02 in benchmark-cases.md |
| **注释缺失** | 任一字段注释为空 | 拒绝生成，提示先补充字段注释 | B-02 in benchmark-cases.md |
| **无主键表** | 表未定义主键字段 | 可继续，但需提示"将按首字段兜底为主键" | B-03 in benchmark-cases.md |
| **前端类型冲突** | application.yaml 默认 front-type 与用户目标栈不一致 | 需澄清目标 front-type，必要时拒绝走主路径 | B-04 in benchmark-cases.md |
| **输入无变化** | 输入 schema 与现有生成目标没有有效差异 | 拒绝重复生成，提示"输入未变化"并转模板预览/复用现有结果 | B-05 in benchmark-cases.md |
| **树表字段缺失** | 树表缺少 treeParentColumnId 映射或缺少 treeNameColumnId 映射 | 拒绝走主路径，提示先补树字段或改回单表 | B-06 in benchmark-cases.md |
| **主子表关系缺失** | 主表生成时没有子表，或 subJoinColumnId 无效 | 拒绝走主路径，提示先补子表关系配置 | B-07 in benchmark-cases.md |
| **主子表关系缺失** | 主子表场景下子表或关联字段（subJoinColumnId）缺失/无效 | 拒绝走主路径，先补子表关系配置，或改为单表生成 | B-08 in benchmark-cases.md |

- 通过阈值：以上 9 行边界场景至少覆盖 `7/9`，且"注释缺失、树表字段缺失、主子表关系缺失"3 类必须给出停止或告警处理，不能误导为可直接继续生成。
- 评分切线：`100 = 9/9`；`80 = 7/9` 且硬边界（注释缺失、树表字段缺失、主子表关系缺失）全命中；`0 = <= 6/9` 或任一硬边界处理失当。

---

## 评分汇总表

| 维度 | 权重 | 得分 | 备注 |
|------|------|------|------|
| 触发准确性 | 20% | /100 | |
| 仓库识别 | 15% | /100 | |
| 场景分类 | 15% | /100 | |
| 工作流正确性 | 25% | /100 | |
| 前端类型映射 | 10% | /100 | |
| 边界纪律 | 15% | /100 | |
| **总分** | 100% | /100 | |

- 建议总评 Pass 线：总分 `>= 80`，且"触发准确性、工作流正确性、前端类型映射"三个维度均单独通过。

---

## 使用方式

1. **人工复核** - 评审员根据量规逐项打分，填写汇总表
2. **自动化** - 将量规维度映射为断言，编写脚本批量检查
3. **迭代优化** - 针对低分维度重写对应 SKILL.md 章节
