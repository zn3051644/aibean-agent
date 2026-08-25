---
name: aibean-business-development
description: "在用户指定的 AiBean Vue 前端与 Java Biz API 项目中开发业务列表、业务表单和 CRUD 接口；流程需求仅复用既有流程编码与元数据，不设计 BPMN 流程图。"
---

# Aibean Business Development

在用户指定的本地前端与后端工作区中，交付可运行的业务功能。以当前项目源码为准；本技能中的路径和协议是 AiBean 的默认约定，而不是对任意项目的硬编码。

## 先决条件与范围

- 若用户未提供前端、后端本地目录，先只询问这两个目录。确认前端存在 `package.json` 和 `src/`，后端存在根 `pom.xml` 及 `aibean-biz-web` 模块；不能假定固定绝对路径。
- 覆盖普通业务 CRUD、流程关联业务的业务数据/表单/列表，以及对既有业务页面的扩展。
- 不设计 BPMN 图、不新建待办/审批/提交/驳回等流程页面或流转逻辑。流程入口确有需要时，仅复用已有前端能力。
- 业务 Java 代码统一放入 `aibean-biz-web`，不为业务功能新建 Maven 模块。
- 只在用户已给权限的本地工作区中读写；不要把凭据、令牌或本地环境配置写入源码、文档或提交记录。
- 所有在线 AiBean API（包括 Server API 的 `/api` 与 Biz API 代理的 `/bapi`）调用均需要认证。调用前读取 [临时 API 认证](references/authentication.md)：仅在当前会话获得用户输入的临时 `Authorization` 值，未认证时不调用在线 API。

## 工作方式

1. 先阅读与当前需求直接相关的现有页面、API、实体和配置；不要用示例中的表名、流程编码、IP 或账号作为新功能的默认值。开发列表时读 [前端列表约定](references/frontend-lists.md)，开发 Java 接口时读 [后端约定](references/backend-development.md)。
2. 判断需求是“既有表扩展”“新表业务”还是“流程关联业务”。仅在缺少会影响数据模型、流程编码或菜单名称的信息时提问，并且一次只问一个问题。
3. 需要调用在线 AiBean API 时，先确认当前会话已有可用认证；没有时请求用户输入临时令牌。认证失败（401/403）后停止在线调用并要求用户重新登录/提供新令牌，不得猜测、重试旧令牌或跳过认证。
4. 新表、字段新增/删除/类型变更必须先让用户确认表和字段。已有物理表（尤其是流程绑定表）只做映射和读写业务逻辑；不得用 AutoTable 修改它。
5. 需要菜单时，先读取已有菜单树推断父级、编码和排序；在写入 `menu/admin/save` 前只确认菜单名称，然后再创建或更新菜单。不要单独实现权限功能。
6. 流程表或默认表单需要在线绑定时，先完成本地文件或表结构的核对，再展示将发送的请求体并取得用户对这一次写入的明确确认。完成后说明改动文件、验证结果、菜单结果和流程绑定结果。

## 流程关联业务

用户提供流程编码后，先通过 Server API 查询 `GET /api/process/meta/bound-fields?processCode=...`；必要时可带 `processVersion`。在前端开发代理下使用相对 `/api` 路径；不要改用旧的 `/bapi/process/meta/bound-fields`。读取 [流程元数据](references/process-metadata.md) 后再生成数据访问、列表和表单。

- 查询失败、没有绑定表，或元数据不足以支持需求时，停止推断并向用户逐项确认表/字段。
- 该接口返回的表和字段是既有流程设计的事实来源：不改 BPMN；已绑定表不改物理表。
- 每个流程数据表必须包含 `itemid` 与 `taskid` 两个 `long` 字段：`itemid` 是且只能是唯一主键，`taskid` 由流程引擎写入，不得设为第二主键或由业务代码主动写入。具体映射规则见 [流程元数据](references/process-metadata.md)。
- 需要新增、更新或移除流程表绑定，或者设置流程默认表单时，读取 [流程绑定配置](references/process-bindings.md)。两个 POST 写入都必须逐次展示请求体并获得明确确认。
- 表单开发时读 [流程表单约定](references/process-forms.md)。具体 `.vue` 文件只直接创建或修改在前端仓库 `public/form/forms/`，不能调用表单文件管理接口。流程默认表单需要绑定时，在文件保存并核对相对路径后，通过上述配置 API 更新。

## 交付与验证

- 列表页面以 `src/views/panel/demo/demoApp.vue` 的 `query-custom-panel + vxe-grid` 组合为优先范式；不要保留 mock 数据或 mock 查询函数。
- 后端接口使用项目统一响应和异常机制，前端请求封装在 `src/api/`，并根据变更面运行最小且有意义的前端类型检查/构建与 Maven 编译检查。详见 [验证清单](references/verification.md)。
- AutoTable 配置当前可能允许删除列。除非用户明确确认字段变更和风险，不得通过移除/改变 AutoTable 注解来删除或收窄列。
- 不因本技能默认创建 Git 提交、推送、流程定义或数据库视图；这些动作需要用户明确请求或确认。
