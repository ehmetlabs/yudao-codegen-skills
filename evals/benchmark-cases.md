# Benchmark Cases

本文件定义 Yudao-Codegen-Skills 的基准测试用例分组，用于验证 skill 触发准确率与场景覆盖度。

## 测试用例分组

### 1. 正向用例 (Positive)

触发信号明确，应正确识别为 yudao-codegen 工作流的请求。

| ID | 用户输入示例 | 预期结果 | 覆盖场景 | 事实源 |
|----|-------------|---------|---------|-------|
| P-01 | "帮我用 yudao 的 codegen 模板生成一个用户管理模块的 CRUD 代码" | 应触发：单表模板主路径 | 明确提及 yudao + codegen 模板 + CRUD | `CodegenEngine.java` (`SERVER_TEMPLATES`)；`https://doc.iocoder.cn/new-feature/` |
| P-02 | "在 RuoYi-Vue-Pro 里创建一个订单表的单表生成" | 应触发：RuoYi-Vue-Pro / Yudao 单表生成 | 提及 RuoYi-Vue-Pro + 单表 | `*/src/views/infra/codegen/index.vue`；`https://doc.iocoder.cn/new-feature/` |
| P-03 | "使用 infra 模块的 code generator，直接按模板生成表对应代码" | 应触发：收集表结构 -> 推导默认值 -> 渲染模板 | 明确 infra/codegen 模板路径 | `CodegenEngine.java`；`CodegenBuilder.java` (`buildTable` `buildColumns`) |
| P-04 | "我要做一个树表结构的部门管理，有 parentId 层级" | 应触发：树表分支 | 树表场景 + 层级字段 | `CodegenEngineVue3Test.java` (`testExecute_vue3_tree`)；`https://doc.iocoder.cn/new-feature/tree/` |
| P-05 | "主子表生成：订单主表 + 订单明细子表" | 应触发：主子表分支 | 主子表明确提及 | `CodegenEngineVue3Test.java` (`testExecute_vue3_master_*`)；`https://doc.iocoder.cn/new-feature/master-sub/` |
| P-06 | "在 ruoyi-vue-pro 项目里先看 codegen 模板会生成哪些文件，再决定是否落盘" | 应触发：模板预览/输出清单路径 | 预览生成结果 + 项目名 | `CodegenEngine.java`（模板路径到输出路径映射）；`*/src/views/infra/codegen/index.vue` |
| P-07 | "表结构改了，按最新字段重新套用 codegen 模板生成" | 应触发：重新读取表结构后再渲染模板 | 基于最新 schema 重新生成 | `CodegenBuilder.java`；`CodegenServiceImpl.java` (`validateTableInfo`) |
| P-08 | "在 yudao 项目里用 Vue3 Element Plus 模板生成用户管理模块的前端代码" | 应触发：前端类型为 Vue3 Element Plus | 明确 yudao 上下文 + 前端类型 | `CodegenFrontTypeEnum.java` (`VUE3_ELEMENT_PLUS=20`)；`application.yaml` (`yudao.codegen.front-type: 20`)；`CodegenEngineVue3Test.java` |
| P-09 | "字段 `pay_status` 已配置 `dictType=pay_status`，按真实语义生成表单和查询条件" | 应触发：优先采用显式元数据语义，生成 dict-backed 控件与匹配查询行为 | 语义证据完整（显式元数据） | `CodegenColumnDO.java`（`dictType/htmlType/listOperationCondition`）；`CodegenBuilder.java`（fallback 对照） |

### 2. 负向用例 (Negative)

明显不属于 yudao-codegen 工作流，应避免误触发。

| ID | 用户输入示例 | 预期结果 | 排除原因 | 事实源 |
|----|-------------|---------|---------|-------|
| N-01 | "用 MyBatis-Plus 生成 Entity 代码" | 不应触发；拒绝走主路径：这是其他生成器链路 | 明确其他技术栈 | `CodegenEngine.java`（仅覆盖 yudao codegen 模板族）；`https://doc.iocoder.cn/new-feature/` |
| N-02 | "手写一个 Spring Boot Controller" | 不应触发；拒绝走主路径：用户要求手写，不是 codegen | 明确手写而非生成 | `CodegenEngine.java`（模板渲染路径）；`https://doc.iocoder.cn/new-feature/` |
| N-03 | "用 JHipster 创建微服务模块" | 不应触发；拒绝走主路径：这是其他脚手架 | 其他生成器 | `CodegenEngine.java`；`https://doc.iocoder.cn/new-feature/`；`https://doc.iocoder.cn/new-feature/master-sub/` |
| N-04 | "优化数据库查询性能" | 不应触发；拒绝走主路径：与模板渲染无关 | 与 codegen 无关 | `CodegenEngine.java`；`CodegenBuilder.java` |
| N-05 | "配置 Spring Security 权限" | 不应触发；拒绝走主路径：纯权限配置任务 | 纯权限配置 | `CodegenEngine.java`；`*/src/views/infra/codegen/index.vue`（仅为辅助识别信号） |
| N-06 | "用 Python 写一个数据导入脚本" | 不应触发；拒绝走主路径：完全不同技术栈 | 完全不同技术栈 | `CodegenEngine.java`；`https://doc.iocoder.cn/new-feature/` |
| N-07 | "设计数据库 ER 图" | 不应触发；拒绝走主路径：仍处于建模设计阶段 | 设计阶段，非生成阶段 | `https://doc.iocoder.cn/new-feature/`（先有表结构再导入）；`CodegenBuilder.java` (`buildTable`) |
| N-08 | "部署应用到生产环境" | 不应触发；拒绝走主路径：部署不属于 codegen | 部署操作 | `CodegenEngine.java`；`*/src/views/infra/codegen/index.vue` |

### 3. 澄清用例 (Clarification)

信息不足或存在歧义，需要向用户确认后才能继续。

| ID | 用户输入示例 | 预期结果 | 缺失信息 | 澄清问题 | 事实源 |
|----|-------------|---------|---------|---------|-------|
| C-01 | "帮我生成一个模块" | 需澄清：先补表名或模块名 | 未指定表/模块名 | "请提供要生成的数据库表名或模块名称" | `CodegenBuilder.java`（表名推导 `moduleName` / `businessName`）；`https://doc.iocoder.cn/new-feature/` |
| C-02 | "用 codegen 做树表" | 需澄清：先补树字段映射 | 未指定树结构字段 | "请说明当前配置中哪个字段映射为树父字段（treeParentColumnId），哪个字段映射为树名称字段（treeNameColumnId）" | `CodegenEngineVue3Test.java` (`testExecute_vue3_tree`)；`https://doc.iocoder.cn/new-feature/tree/` |
| C-03 | "生成主子表" | 需澄清：先补主表/子表/关联字段 | 未指定子表信息 | "请提供主表名、子表名，以及子表关联主表的字段" | `CodegenServiceImpl.java`（子表与 `subJoinColumnId` 校验）；`https://doc.iocoder.cn/new-feature/master-sub/` |
| C-04 | "我要用 yudao 生成代码" | 需澄清：检查 repo/config 证据后，若前端类型仍无法确定则询问 | 未说明前端类型，且无明确 repo/config 信号 | "请检查 `application.yaml` 中的 `yudao.codegen.front-type` 配置，或确认目标前端栈：Vue3 Element Plus / Vben / Uniapp / 其他？" | `CodegenFrontTypeEnum.java`；`application.yaml` (`front-type`)；`SKILL.md` "Front-type mapping" 检测优先级 |
| C-05 | "按最新表结构重新生成" | 需澄清：先确认最新 schema 来源 | 未指定以数据库、DDL 还是现有元数据为准 | "请提供最新的表结构来源（数据库定义、DDL、字段清单），我再按模板重生" | `CodegenBuilder.java`；`CodegenServiceImpl.java` (`validateTableInfo`) |
| C-06 | "审阅将要渲染的输出结果" | 需澄清：先确认输出目标 | 未指定哪张表或哪种模板分支 | "请提供要审阅的表名，以及它属于单表、树表还是主子表，我再说明会渲染哪些目标文件" | `CodegenEngine.java`；`*/src/views/infra/codegen/index.vue`（辅助信号） |
| C-07 | "生成 CRUD" | 需澄清：先确认单表/树表/主子表 | 未区分单表/树表/主子表 | "请确认生成场景：单表、树表（有层级）还是主子表？" | `CodegenEngineVue3Test.java`（单表/树表/主子表测试）；`https://doc.iocoder.cn/new-feature/`；`https://doc.iocoder.cn/new-feature/tree/`；`https://doc.iocoder.cn/new-feature/master-sub/` |
| C-08 | "直接生成代码" | 需澄清：先确认目标表与模板分支 | 未指定表和场景 | "请提供要生成代码的表名，以及它属于单表、树表还是主子表" | `CodegenEngine.java`；`https://doc.iocoder.cn/new-feature/` |
| C-09 | "这个 `status` 字段按语义帮我生成" | 需澄清：先补 `dictType` 或语义证据来源 | 仅有字段名，缺显式字典/注释语义 | "请确认该字段使用的字典编码（dictType）或补充字段注释语义；若无证据我只能按 fallback 规则处理" | `CodegenBuilder.java`（后缀 fallback）；`SKILL.md`（语义优先级与低置信澄清） |

### 4. 边界用例 (Boundary)

极端情况或边界条件，验证 skill 的健壮性。

| ID | 场景描述 | 边界条件 | 预期处理 | 事实源 |
|----|---------|---------|---------|-------|
| B-01 | 表名不符合 `module_business` 约定 | 无法稳定推导模块名/业务名 | 拒绝走主路径，先规范表名再导入 | `CodegenBuilder.java`（`moduleName` / `businessName` 推导）；`https://doc.iocoder.cn/new-feature/` |
| B-02 | 表或字段注释缺失 | `tableComment` 为空或任一字段 comment 为空 | 拒绝走主路径，先补表注释/字段注释 | `CodegenServiceImpl.java` (`validateTableInfo`)；`https://doc.iocoder.cn/new-feature/` |
| B-03 | 无主键表 | 表没有主键字段 | 可继续，但要提示“将按首字段兜底为主键” | `CodegenServiceImpl.java`（无主键时首字段兜底）；`CodegenBuilder.java` (`buildColumns`) |
| B-04 | 默认前端类型与仓库目标不一致 | `application.yaml` 默认值与用户目标栈冲突 | 需澄清目标 front-type，必要时拒绝走主路径 | `application.yaml` (`front-type: 20`)；`CodegenFrontTypeEnum.java` |
| B-05 | 输入 schema 与现有生成目标没有任何有效差异 | 重新生成不会带来文件级变化 | 拒绝重复生成，提示“输入未变化”，改为检查模板分支或直接复用现有结果 | `CodegenBuilder.java`；`CodegenEngine.java` |
| B-06 | 树表缺少父级/树名关键信息 | 缺少树父字段映射或树名称字段映射 | 拒绝走主路径，先补树字段或改回单表 | `CodegenEngineVue3Test.java` (`testExecute_vue3_tree`)；`https://doc.iocoder.cn/new-feature/tree/` |
| B-07 | 主子表缺少子表或关联字段 | 主表生成时没有子表，或 `subJoinColumnId` 无效 | 拒绝走主路径，先补子表关系配置 | `CodegenServiceImpl.java`（主子表与关联字段校验）；`https://doc.iocoder.cn/new-feature/master-sub/` |
| B-08 | 主子表关系配置缺失 | 子表未绑定主表，或 `subJoinColumnId` / 子表关联字段缺失/无效 | 拒绝走主路径，先补主子表关系配置，或改为单表生成 | `CodegenServiceImpl.java`（主子表与关联字段校验）；`CodegenEngineVue2Test.java` (`testExecute_vue2_master_*` 证明 Vue2 支持主子表) |
| B-09 | 语义证据冲突 | 用户显式要求与字段注释/元数据语义相互冲突，或 `dictType` 缺证据却强制字典控件 | 拒绝直接覆盖，输出冲突证据链并要求最小澄清 | `SKILL.md`（语义优先级与冲突处理）；`CodegenColumnDO.java`（语义字段承载） |

## 使用说明

1. **触发测试**: 使用本文件的 P-01~P-09 验证 skill 是否正常触发
2. **防误触测试**: 使用 N-01~N-08 验证不会误触发
3. **对话补全**: 使用 C-01~C-09 验证 skill 会主动询问缺失信息
4. **健壮性**: 使用 B-01~B-09 验证极端情况的处理

## 扩展指南

新增测试用例时，请遵循以下命名与分组规范：

- 正向用例: P-XX (Positive)
- 负向用例: N-XX (Negative)
- 澄清用例: C-XX (Clarification)
- 边界用例: B-XX (Boundary)
