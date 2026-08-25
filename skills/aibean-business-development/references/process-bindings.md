# 流程绑定配置

当流程业务需要让平台自动识别新建/变更的数据表，或把本地 Vue 表单设为当前流程的默认表单时，使用本说明。它不用于设计 BPMN 图或实现流程流转页面。

## 共同约束

- 调用前遵循 [临时 API 认证](authentication.md)。`/api` 与 `/bapi` 使用同一个临时 `Authorization` 值。
- 在前端开发环境使用相对 `/api` 请求路径，由现有代理转发；不要硬编码 Server API 主机地址。
- 两个 POST 都是远程配置写入：先完成本地表/表单核对，展示本次将发送的完整 JSON 请求体，并取得用户对这一次写入的明确确认后才能调用。
- 成功后重新查询 `GET /api/process/meta/bound-fields?processCode=<流程编码>`，或读取接口响应，核对写入结果；失败时停止并报告服务返回，不猜测是否已生效。

## 流程发起表

使用：

~~~text
POST /api/process/initiator-table/operate
~~~

请求顶层包含 `processCode` 与 `operations`。每个操作至少使用已确认的 `operation`、`tableName`；`ADD` 和 `UPDATE` 必须提供 `isDetail`，其中 `false` 为主表、`true` 为明细表。一次请求可包含主表和多个明细表操作。

~~~json
{
  "processCode": "<已确认的流程编码>",
  "operations": [
    {
      "operation": "ADD",
      "tableName": "<已确认的物理表名>",
      "isDetail": false
    }
  ]
}
~~~

- `operation: "default"` 用于数据源配置。现有接口资料未列出其完整附加字段时，必须先读取用户项目中的 OpenAPI 或服务端实现再构造请求；不得猜测字段。
- `DELETE` 仅解除流程配置，不删除物理表或业务数据。不得把它作为清理表或回滚失败操作的默认动作；仅在用户明确要求解除绑定、展示请求体并再次确认后调用。
- 对已有流程绑定表只做映射，不用 AutoTable 改动它。对于新表，先确认表与字段、落实 `itemid`/`taskid` 约定并由用户重启 Biz API 建表，再提交 `ADD` 或 `UPDATE` 绑定。

## 流程默认表单

在本地保存 `public/form/forms/<relative-path>.vue` 后，使用：

~~~text
POST /api/process/default-form/update
~~~

~~~json
{
  "processCode": "<已确认的流程编码>",
  "defaultForm": "<不含 .vue 的已核对相对路径>"
}
~~~

此接口仅更新当前 ACTIVE 版本的默认表单。除非用户明确要求清空绑定，`defaultForm` 必须是已存在且已核对的非空相对路径；空字符串会清空当前流程默认表单。调用成功后交付相对路径、发送结果和验证状态，不再要求用户到设计器手工绑定该默认表单。
