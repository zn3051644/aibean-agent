# AiBean 业务开发技能设计

## 目标

`aibean-business-development` 是一个可由 Codex 或 deepseek-harness 加载的通用 `SKILL.md` 技能。它让智能体在用户明确提供的本地 AiBean 前端与 Biz API 工作区中，快速交付业务 CRUD、查询列表和流程关联表单，而不把流程平台本身重新实现一遍。

## 边界

| 范围内 | 明确排除 |
| --- | --- |
| Java 业务接口、数据映射、列表页面、动态菜单、流程业务表单，以及流程表/默认表单的 API 绑定 | BPMN 图设计、审批/待办/提交/驳回页面、流程流转实现 |
| 已有表映射与新业务表 AutoTable 建表 | 对既有流程表做 AutoTable DDL 修改 |
| 通过 Server API 查询/维护流程绑定元数据 | 绕过认证、硬编码 Server API 地址或使用旧 BAPI 元数据代理 |

## 运行模型

~~~text
自然语言业务需求
        |
        +-- 流程编码 --> /api/process/meta/bound-fields --> 既有表映射 + 表单/列表
        |                         |
        |                         +-- 表结构确认 + 本次写入确认
        |                         `--> /api/process/initiator-table/operate --> 回查元数据
        |                                                               |
        |                           本地 public/form/forms/*.vue ------+
        |                                                               `--> /api/process/default-form/update
        |
        +-- 新业务数据 --> 用户确认表字段 --> AutoTable 实体 --> 用户重启 Biz API 建表
        |
        +-- 前端页面 --> 查询面板 + vxe-grid / public/form/forms/*.vue
        |
        `-- 菜单名称确认 --> /bapi/menu/admin/save --> 动态路由
~~~

## 源码依据

- 列表范式：`web/src/views/panel/demo/demoApp.vue`。
- 表单运行壳：`web/src/views/form/formvue/vform.vue`；模板、组件和组件配置分别在 `web/public/form/forms/templete.vue`、`web/public/form/components/`、`web/public/form/formConfig.json`。
- Java 业务模块：`aibean-biz/aibean-biz-web`，使用 Spring Boot、MyBatis-Flex、AutoTable 与统一 `R` 响应。
- 流程元数据：`GET /api/process/meta/bound-fields`；返回流程、表、字段、主键和明细表信息。
- 流程绑定：`POST /api/process/initiator-table/operate`；默认表单：`POST /api/process/default-form/update`。
- 菜单代理：`POST /bapi/menu/admin/save`，路由由 `GET /bapi/menu/routes` 提供。

## 决策和安全门槛

智能体在未获得前端/后端目录前不修改代码。流程业务先读取 Server API 元数据；若元数据缺失或与需求冲突，则向用户逐项确认。新增、删除或变更字段必须获得确认；流程已绑定表一律禁止 AutoTable。写数据库菜单前确认菜单名称。写入流程表绑定或默认表单前，必须展示完整请求体并取得当次用户确认；成功后重新查询或核对接口响应。

流程数据表统一使用 `long itemid` 与 `long taskid`：`itemid` 是每张表唯一且唯一允许的主键；`taskid` 是由流程引擎写入的任务 ID，不能设为第二主键，也不能由业务页面或 CRUD 主动写入。

所有在线 AiBean API 均要求认证。Server API 的 `/api` 与 Biz API 代理的 `/bapi` 共用同一个令牌。技能在调用前请求用户通过安全输入提供当前登录态的临时 `Authorization: Bearer <token>` 值，并只在当前会话内使用。令牌不写入源码、配置、文档、Git、URL 或日志；401/403 时停止在线调用并要求用户重新登录。没有令牌时，智能体仍可以完成本地代码工作，但不得伪造在线查询或写入结果。

## 仓库布局

~~~text
aibean-agent/
├── skills/
│   └── aibean-business-development/
│       ├── SKILL.md
│       ├── agents/openai.yaml
│       └── references/
└── docs/
    └── aibean-business-development-design.md
~~~

这个布局允许后续在 `skills/` 下增加互不耦合的技能。每个技能把具体运行约定放到自己的 `references/`，避免把平台经验复制到所有技能中。
