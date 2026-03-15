# Scenario Matrix

概览代码生成核心场景的触发/线索与已验证来源。

scenario | trigger / cue | source of truth | notes
- | - | - | -
单表 CRUD 生成 | 选择 `CodegenTemplateTypeEnum.ONE` 并按 `SERVER_TEMPLATES`/`FRONT_TEMPLATES` 渲染标准单表文件集 | CodegenTemplateTypeEnum.java | Agent 直接套用单表模板，不依赖导入/预览/下载 API
树表生成 | 选中树表模板后要求 `treeParentColumnId`/`treeNameColumnId` 并走专属 tree 模板 | https://doc.iocoder.cn/new-feature/tree/ | 文档与模板共同定义树表工作流区别于单表
主子表生成 | 主表模板拉取 `SUB` 子表模板并校验关联字段后一起渲染 | CodegenServiceImpl.java | 主表生成前需保障子表存在且 join 字段合法
模板渲染/输出路径 | `CodegenEngine` 根据 `SERVER_TEMPLATES` / `FRONT_TEMPLATES` 计算输出文件路径 | CodegenEngine.java | 结果是文件路径 → 代码内容映射，可由 agent 直接写入目标仓库
字段校验/默认推导 | `validateTableInfo` + `CodegenBuilder.buildTable/buildColumns` 提供表/字段约束和默认值 | CodegenServiceImpl.java | 注释完整性、命名、UI 类型和示例值都来自源码规则
Builder 默认模板 | `CodegenBuilder.initTableDefault` 将 templateType 设为 `ONE`，并推导模块/类名 | CodegenBuilder.java | 处理 module/business/class/comment 默认值
前端类型/配置基线 | `CodegenFrontTypeEnum` 提供可选前端模板列表，`application.yaml` 提供默认 front-type | CodegenFrontTypeEnum.java | 默认 `yudao.codegen.front-type: 20`，但 agent 仍需结合 repo 结构判断
管理台入口/官方文档 | `index.vue` 中 4 个 `doc-alert` 指向单表/树表/主子表/单元测试 官方页 | */src/views/infra/codegen/index.vue | UI 与文档是辅助识别信号，不是生成主路径
