---
name: yudao-codegen-skills
description: Trigger on Yudao / RuoYi-Vue-Pro codegen, generate, CRUD, scaffold workflows. Agent reads codegen templates, builder defaults, enums, and config to render code directly from the repo template set. Covers 单表、树表、主子表 and front-type mappings without relying on `/infra/codegen` APIs.
---
# yudao-codegen-skills

Code generation skill for Yudao / RuoYi-Vue-Pro family repositories. Agent inspects the repo's `codegen` template resources, builder rules, engine mappings, and enum/config definitions, then renders scaffolded CRUD code directly.

## Prerequisites

- Repository belongs to Yudao / RuoYi-Vue-Pro family
- `yudao-module-infra` module present
- `src/main/resources/codegen/` template directory exists
- Database tables already created and accessible

## When to trigger

Trigger this skill when user requests:

- **codegen / generate / CRUD / scaffold** for Yudao / RuoYi-Vue-Pro
- **树表** (tree table) code generation
- **主子表** (master-sub table) code generation
- **单表** (single table) standard CRUD generation
- Use repo `codegen` templates to directly render CRUD code
- Infra module codegen UI（例如 `*/src/views/infra/codegen/index.vue`）

**Over-trigger guard**: Do NOT trigger for generic CRUD or other frameworks (Spring Initializr, JHipster, etc.). Only for Yudao / RuoYi-Vue-Pro codegen workflows.

## Repository detection

Presence of these confirms Yudao codegen capability:

1. **Template resources**: `src/main/resources/codegen/` contains Java / SQL / Vue / Vue3 / Vben / Uniapp template families

2. **Engine/Builder**: `CodegenEngine.java` and `CodegenBuilder.java`
   - `CodegenEngine` defines `SERVER_TEMPLATES` and `FRONT_TEMPLATES`
   - `CodegenBuilder` derives `moduleName`, `businessName`, `className`, default `templateType`, and column UI/operation defaults

3. **Enums**: `CodegenTemplateTypeEnum.java` (ONE, TREE, MASTER_NORMAL, MASTER_ERP, MASTER_INNER, SUB), `CodegenFrontTypeEnum.java` (VUE2_ELEMENT_UI, VUE3_ELEMENT_PLUS, VBEN variants, VUE3_ADMIN_UNIAPP_WOT)

4. **Config**: `application.yaml` with `yudao.codegen.front-type`, `vo-type`, `delete-batch-enable`, `unit-test-enable`

5. **Supporting signals only**: optional admin UI/docs such as `*/src/views/infra/codegen/index.vue` and official docs links; these help identify the repo family but are NOT the generation path

## Preflight checklist

Before executing codegen workflow, verify ALL of the following:

### Schema Input Checks (Hard Stops)
- [ ] **表名** follows naming convention: `moduleName_businessName` (e.g., `system_user`, `bpm_process`)
- [ ] **表注释** is set and descriptive (required by `validateTableInfo`, generates class comment)
- [ ] **字段注释** are set for ALL columns (required by `validateTableInfo`, and should exist in the schema input the agent uses)
- [ ] **主键** is defined (if missing, first field becomes primary key fallback)

### Configuration Checks
- [ ] **Module name** determined (affects Java package: `cn.iocoder.yudao.module.{moduleName}`)
- [ ] **Business name** determined (affects class naming: `{BusinessName}Service`, `{BusinessName}Controller`)
- [ ] **Template type** selected: `ONE(1)`, `TREE(2)`, `MASTER_NORMAL(10)`, `MASTER_ERP(11)`, `MASTER_INNER(12)`, or `SUB(15)`
- [ ] **Front type** confirmed: defaults to `VUE3_ELEMENT_PLUS(20)` from `yudao.codegen.front-type` in `application.yaml`
- [ ] **Parent menu ID** set (for UI navigation menu placement)

### STOP Conditions - 停止生成 - Do NOT proceed if:
1. **表注释为空** → 停止生成 and add table comment first: `ALTER TABLE xxx COMMENT = '...'`
2. **字段注释为空** → 停止生成 and add column comments: `ALTER TABLE xxx MODIFY COLUMN xxx COMMENT = '...'`
3. **Template type mismatch** → 停止生成 and verify table structure matches scenario (tree needs `treeParentColumnId` and `treeNameColumnId` configured, master-sub needs foreign keys)
4. **Schema source不完整或冲突** → 停止生成 - if the agent cannot determine field comments, join fields, or target stack from repo/config evidence, ask user before rendering

## Workflow

### Template-Driven Main Process

The standard template-driven codegen execution flow:

```
[识别仓库信号] → [读取 codegen 模板目录与映射] → [检查 yudao.codegen / front-type] → [执行前置检查]
       ↓
[收集表结构输入] → [推导默认值] → [选择模板分支] → [渲染模板] → [校验输出路径] → [落盘/整理]
```

#### Phase 1: Pre-execution Checks
1. **Verify Repository Context**
   - Confirm `src/main/resources/codegen/` exists and contains the expected template families
   - Confirm `CodegenEngine.java` exposes `SERVER_TEMPLATES` / `FRONT_TEMPLATES`
   - Check `application.yaml` for `yudao.codegen.front-type` (default: 20 = VUE3_ELEMENT_PLUS)

2. **Validate Table Readiness** (Critical - Hard Stop if Failed)
   - Execute: `validateTableInfo()` logic equivalent
   - Verify: Table has comment, ALL columns have comments
   - **STOP if**: `CODEGEN_TABLE_INFO_TABLE_COMMENT_IS_NULL` or `CODEGEN_TABLE_INFO_COLUMN_COMMENT_IS_NULL`

#### Phase 2: Collect Schema Input (收集表结构输入)

```java
// Direct template-driven flow:
CodegenBuilder.buildTable(tableInfo)            // Creates CodegenTableDO defaults
  → CodegenBuilder.buildColumns(tableId, fields) // Creates CodegenColumnDO defaults
  → choose templateType/frontType
  → CodegenEngine.execute(dbType, table, columns, subTables, subColumnsList)
```

**Key behaviors**:
- Module name: extracted from table prefix (e.g., `system_user` → `system`)
- Business name: extracted from table suffix (e.g., `system_user` → `user`)
- Class name: CamelCase of business name (e.g., `user` → `User`)
- Primary key: if not defined, first column becomes PK
- Default template type: `ONE(1)`
- Default front type: from `codegenProperties.getFrontType()`

#### Phase 3: Derive Defaults and Configure Metadata (推导默认值)

**Builder behavior**:
```java
initTableDefault(table)
→ moduleName = prefix before first '_'
→ businessName = camelCase(after first '_')
→ className = UpperCamelCase(after first '_')
→ classComment = removeSuffix(tableComment, "表")
→ templateType = ONE by default

buildColumns(tableId, tableFields)
→ processColumnOperation(column)
→ processColumnUI(column)
→ processColumnExample(column)
```

**STOP conditions for metadata derivation**:
- If naming convention is too ambiguous to derive stable `moduleName` / `businessName`, stop and ask user
- If tree/master-sub required fields are missing, stop and correct schema inputs first

#### Phase 4: Render Templates (渲染模板)

**Generation engine**: `CodegenEngine.execute()`
- Uses Velocity templates
- Template selection based on `CodegenTemplateTypeEnum`
- Frontend template group based on `CodegenFrontTypeEnum`
- Output path selection from `SERVER_TEMPLATES` / `FRONT_TEMPLATES`
- Backend mapper XML must be rendered from `codegen/java/dal/mapper.xml.vm` and written under `src/main/resources/mapper/**`

**Files generated**:
- Java: Controller, Service, ServiceImpl, Mapper, DO, DTO/VO
- MyBatis XML: Mapper XML generated from `mapper.xml.vm` (at least main table; for master-sub, include sub-table mapper XML to keep CRUD artifacts complete)
- Vue: views, API, components
- SQL: menu inserts
- Test: Unit tests (if enabled)

#### Phase 5: Review Output Set and Write Files (落盘/整理)

**Post-generation minimal organization**:
1. Review the generated file path → content map
2. Place Java files in appropriate module structure
3. Place MyBatis XML mapper files in `src/main/resources/mapper/{businessName}/{ClassName}Mapper.xml` (master-sub: include sub-table mapper XML under its own business directory)
4. Place Vue files in your admin frontend project (for example `*/src/views/`)
5. Keep SQL/menu artifacts aligned with the selected scenario and front type
6. Avoid writing files that do not belong to the detected repo variant

### Failure & Recovery Branches

#### Branch 1: Missing Table Comment
**Detection**: `validateTableInfo()` throws `CODEGEN_TABLE_INFO_TABLE_COMMENT_IS_NULL`

**Action**:
```sql
ALTER TABLE your_table COMMENT = 'Your table description';
```
**Then**: Re-run import

#### Branch 2: Missing Column Comments
**Detection**: `validateTableInfo()` throws `CODEGEN_TABLE_INFO_COLUMN_COMMENT_IS_NULL`

**Action**:
```sql
ALTER TABLE your_table MODIFY COLUMN column_name data_type COMMENT 'Column description';
```
**Then**: Re-run import

#### Branch 3: Output Stack Conflict
**Detection**: `front-type` config and template/output evidence point to different stacks

**Action**:
1. Stop generation
2. Compare `application.yaml`, `CodegenFrontTypeEnum`, and `CodegenEngine.FRONT_TEMPLATES`
3. Ask user which stack should be treated as source of truth
4. Resume only after stack is explicit

#### Branch 4: Scenario Mismatch
**Detection**: Template type doesn't match table structure

**Examples**:
- Tree table selected but tree parent/name mapping is incomplete (ensure `treeParentColumnId`/`treeNameColumnId` point to actual columns)
- Master-sub selected but foreign key constraints missing

**Action**:
1. Stop generation
2. Either:
   - Fix table structure (configure `treeParentColumnId` and `treeNameColumnId` for tree tables, or add the FK for master-sub), OR
   - Change template type to match actual structure (`ONE` for simple tables)
3. Re-import with corrected settings

### Real Behavior Notes

**From CodegenServiceImpl**:
- `validateTableInfo()`: Strict validation - no empty table/field comments allowed
- Default `frontType`: Comes from `codegenProperties.getFrontType()`, not hardcoded
- Default field inference: From `CodegenBuilder` maps based on field name suffixes

**From CodegenBuilder**:
- Field naming conventions drive defaults:
  - `*name` → `LIKE` query condition
  - `*time`, `*date` → `BETWEEN` query condition
  - `*status`, `*sex` → `RADIO` HTML type
  - `*type` → `SELECT` HTML type
  - `*image` → `IMAGE_UPLOAD`, `*file` → `FILE_UPLOAD`
  - `*content`, `*description` → `EDITOR` HTML type

**From CodegenEngine**:
- Renders file path → code content directly from template maps
- Frontend output paths differ by enum family (`views/*`, `modules/*`, `pages-*`)
- `cloudEnable` changes output module structure for cloud vs monolith repos

## Scenario branches

Use this decision tree to select the correct template type and configuration:

### Decision Tree

```
[开始] → 表是否存在父子层级关系?
    │
    ├─ 是 → 需要树形结构展示?
    │   │
│   ├─ 是 → 【树表 TREE】
│   │       要求: 通过 `treeParentColumnId` + `treeNameColumnId` 配置父节点列和显示名称列（在 `CodegenTableSaveReqVO` 及 UI 表单中按字段编号配置）
│   │       模板: TREE(2)
    │   │
    │   └─ 否 → 按普通单表处理
    │
    └─ 否 → 是否存在一对多关联表?
        │
        ├─ 是 → 选择主子表模式:
        │   │
        │   ├─ 普通弹窗选择模式 → 【MASTER_NORMAL(10) + SUB(15)】
        │   │   适用: 常规主从关系，子表数据量中等
        │   │   交互: 弹窗选择/新建子记录
        │   │
        │   ├─ ERP风格模式 → 【MASTER_ERP(11) + SUB(15)】
        │   │   适用: ERP业务场景，复杂主从关系
        │   │   交互: 类ERP操作界面
        │   │
        │   └─ 内嵌行内编辑模式 → 【MASTER_INNER(12) + SUB(15)】
        │       适用: 子表字段少，快速编辑
        │       交互: 行内直接编辑，无需弹窗
        │
        │   主子表强制校验:
        │   - 子表必须配置外键字段关联主表
        │   - 子表 templateType = SUB(15)
        │   - CodegenServiceImpl.generationCodes 会校验子表存在性和字段映射
        │
        └─ 否 → 【单表 ONE】
            模板: ONE(1)
            默认场景，无特殊关联关系
```

### Branch Details

#### Branch 1: 单表 (ONE)
**Trigger**: 无父子层级，无关联从表
**Template**: `ONE(1)`
**Requirements**: 标准CRUD字段即可
**Default**: 是，最常见场景

#### Branch 2: 树表 (TREE)
**Trigger**: 存在父子层级关系且需要树形展示
**Template**: `TREE(2)`
**Configuration**:
- 通过 `treeParentColumnId` 选定父节点引用列、`treeNameColumnId` 选定展示名称列（在 `CodegenTableSaveReqVO` 及 UI 表单中按字段编号配置）
- `sort` 列经常用于同级排序，作用明显但并非校验硬性要求，未提供也可以生成

**Validation**: `CodegenTemplateTypeEnum.isTree(type)` + `CodegenTableSaveReqVO.isTreeValid()` 确保映射字段齐全
**Behavior**: 生成时排除部分 VO 类，前端使用树形组件

#### Branch 3: 主子表 - 普通模式 (MASTER_NORMAL + SUB)
**Trigger**: 一对多关系，常规弹窗交互
**Templates**: `MASTER_NORMAL(10)` (主表) + `SUB(15)` (子表)
**Interaction**: 弹窗选择/新建子记录
**适用**: 子表数据量中等，需要完整表单校验

#### Branch 4: 主子表 - ERP模式 (MASTER_ERP + SUB)
**Trigger**: ERP业务场景，复杂主从关系
**Templates**: `MASTER_ERP(11)` (主表) + `SUB(15)` (子表)
**Interaction**: 类ERP操作界面，通常有行号、金额合计等
**适用**: 订单、报价单等ERP单据

#### Branch 5: 主子表 - 内嵌模式 (MASTER_INNER + SUB)
**Trigger**: 子表字段少，需要快速行内编辑
**Templates**: `MASTER_INNER(12)` (主表) + `SUB(15)` (子表)
**Interaction**: 行内直接编辑，无需弹窗
**适用**: 子表字段不超过3-5个，追求操作效率

### Master-Sub Validation Rules

`CodegenServiceImpl.generationCodes` enforces:

1. **Sub Table Existence**: If master template type (`isMaster=true`), must have at least one sub table configured
2. **Join Column Validation**: Sub table must have `subJoinColumnId` pointing to valid column in sub table
3. **Foreign Key Constraint**: Sub table column used for join must exist and be properly typed
4. **Template Type Check**: Sub table must have `templateType = SUB(15)`

Failure on any validation → `CODEGEN_MASTER_GENERATION_FAIL_NO_SUB_TABLE` or `CODEGEN_SUB_COLUMN_NOT_EXISTS`

### One-to-Many / Join Column Behavior

From `CodegenServiceImpl.generationCodes`:
- Master table with `isMaster=true` triggers sub-table loading
- Multiple sub-tables supported: `Arrays.asList(contactTable, teacherTable)`
- Each sub-table configures `subJoinColumnId` (which column links to master)
- Each sub-table configures `subJoinMany` (boolean: one-to-many or one-to-one)
- Engine receives: `codegenEngine.execute(dbType, table, columns, subTables, subColumnsList)`

## Front-type mapping

| Enum Constant | Value | Frontend Stack | Notes |
|--------------|-------|----------------|-------|
| `VUE2_ELEMENT_UI` | 10 | Vue 2 + Element UI | Legacy support |
| `VUE3_ELEMENT_PLUS` | 20 | Vue 3 + Element Plus | Modern Element Plus stack |
| `VUE3_VBEN2_ANTD_SCHEMA` | 30 | Vue 3 VBEN2 + ANTD + Schema | Admin pro template |
| `VUE3_VBEN5_ANTD_SCHEMA` | 40 | Vue 3 VBEN5 + ANTD + Schema | Modern variant |
| `VUE3_VBEN5_ANTD_GENERAL` | 41 | Vue 3 VBEN5 + ANTD Standard | Modern variant |
| `VUE3_VBEN5_EP_SCHEMA` | 50 | Vue 3 VBEN5 + EP + Schema | Modern variant |
| `VUE3_VBEN5_EP_GENERAL` | 51 | Vue 3 VBEN5 + EP Standard | Modern variant |
| `VUE3_ADMIN_UNIAPP_WOT` | 60 | Vue 3 Admin + Uniapp + WOT | Mobile admin |

Configuration in `application.yaml`:
```yaml
yudao:
  codegen:
    front-type: 20  # Default fallback (VUE3_ELEMENT_PLUS); do not treat as universal assumption
    vo-type: 10
    delete-batch-enable: true
    unit-test-enable: false
```

Detect the correct front type by combining evidence in priority order:
1. **Explicit `yudao.codegen.front-type`** – honor the configured value when it matches the repo shape. Treat the `20` default as a fallback only after other clues are exhausted.
2. **Template outputs listed in `CodegenEngine.FRONT_TEMPLATES`** – each enum maps to distinct template keys/paths (`codegen/vue3_vben5_antd/schema`, `codegen/vue3_vben5_ele/general`, `codegen/vue3_admin_uniapp`, etc.). Look for frontend directories in the target repo that match expected output patterns (for example `*/src/views/infra/codegen`, `*/pages-*`) and map them to the corresponding front type.
3. **Repo shape and build flags** – inspect whether `cloudEnable` is true (Spring Cloud multi-module) or false (monolith). Combine this with module layout to determine which template families are actually maintained in that repo variant.
4. **Frontend directories and README signals** – examine the target repo frontend README/docs for supported stacks, and check existing frontend directories in workspace. These clues confirm which enum families are alive.
5. **Fallback confirmation** – if none of the above decisively points to a stack, pause and ask the user to state their desired front-type rather than guessing.

Master-sub support caveats:
- VBEN/VBEN5 stacks generate bespoke sub-table components (`form_sub_normal`, `form_sub_inner`, `form_sub_erp`, `list_sub_*`), so ensure the chosen enum matches the existing sub-table UI patterns; otherwise the generated references will point to missing modules.
- Uniapp/mobile stack (`VUE3_ADMIN_UNIAPP_WOT`) writes files under `pages-*/`, so double-check that the mobile frontend directories exist in the repo before selecting enum `60` to avoid generating desktop-only file structures.

**Conflict rule:** if the config claims one front-type (say `front-type: 20`), but you observe frontend directories or template resources matching a different stack family (for example, seeing VBEN-related templates when front-type claims Element Plus, or vice versa), stop and ask the user which stack should be treated as the source of truth before proceeding. Do NOT proceed when `yudao.codegen.front-type` disagrees with the observed template output paths or repo family; require explicit confirmation.

## Do / Don't

**DO:**
- Trigger when user explicitly mentions Yudao / RuoYi-Vue-Pro codegen
- Check template resources as primary signals (`src/main/resources/codegen/`, `CodegenEngine.java`, `CodegenBuilder.java`)
- Use admin UI artifacts (e.g., `*/src/views/infra/codegen/index.vue`) only as optional supporting signals to identify the repo family
- Use official docs for scenario specifics: `https://doc.iocoder.cn/new-feature/` (single table), `https://doc.iocoder.cn/new-feature/tree/` (tree table), `https://doc.iocoder.cn/new-feature/master-sub/` (master-sub table)
- Check `application.yaml` for `yudao.codegen.*` defaults before advising configuration
- Reference `CodegenTemplateTypeEnum` and `CodegenFrontTypeEnum` for template/front-type decisions

**DON'T:**
- Do NOT trigger for generic CRUD requests outside Yudao/RuoYi-Vue-Pro ecosystem
- Do NOT provide template deep customization guidance (Velocity template structure changes are out of scope)
- Do NOT advise on deployment, CI/CD pipeline, or actual database migration execution
- Do NOT assume all CRUD/scaffold requests should use this skill—verify repository context first
- Do NOT modify `examples/scenario-matrix.md` or create `evals/` files in this task

## Validation references

- **Official Documentation**:
  - Single Table: `https://doc.iocoder.cn/new-feature/`
  - Tree Table: `https://doc.iocoder.cn/new-feature/tree/`
  - Master-Sub Table: `https://doc.iocoder.cn/new-feature/master-sub/`
  - Unit Test: `https://doc.iocoder.cn/unit-test/`

- **Key Source Files**:
  - Service: `<target-repo>/yudao-module-infra/src/main/java/cn/iocoder/yudao/module/infra/service/codegen/CodegenServiceImpl.java`
  - Builder: `<target-repo>/yudao-module-infra/src/main/java/cn/iocoder/yudao/module/infra/service/codegen/inner/CodegenBuilder.java`
  - Engine: `<target-repo>/yudao-module-infra/src/main/java/cn/iocoder/yudao/module/infra/service/codegen/inner/CodegenEngine.java`
  - Enums: `<target-repo>/yudao-module-infra/src/main/java/cn/iocoder/yudao/module/infra/enums/codegen/CodegenTemplateTypeEnum.java`, `<target-repo>/yudao-module-infra/src/main/java/cn/iocoder/yudao/module/infra/enums/codegen/CodegenFrontTypeEnum.java`
  - Config: `<target-repo>/yudao-server/src/main/resources/application.yaml` (or `application-dev.yaml`)
- Supporting UI: `*/src/views/infra/codegen/index.vue`

- **Template Resource Anchors**:
  - `src/main/resources/codegen/java/**`
  - `src/main/resources/codegen/sql/**`
  - `src/main/resources/codegen/vue/**`
  - `src/main/resources/codegen/vue3/**`
  - `src/main/resources/codegen/vue3_vben/**`
  - `src/main/resources/codegen/vue3_vben5_antd/**`
  - `src/main/resources/codegen/vue3_vben5_ele/**`
  - `src/main/resources/codegen/vue3_admin_uniapp/**`

- **Supporting UI/API Evidence** (optional, not generation path):
  - `*/src/views/infra/codegen/index.vue`
