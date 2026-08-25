# 流程绑定表元数据

## 查询

当且仅当用户已给出流程编码时，先调用认证后的 Server API：

~~~text
GET /api/process/meta/bound-fields?processCode=<流程编码>
~~~

可选参数 `processVersion` 为空时表示读取命中的流程版本。在前端开发环境中使用相对 `/api` 路径，由用户项目现有的 Vite 代理转发到 Server API；不要硬编码代理目标地址，也不要改用 `/bapi/process/meta/bound-fields`。不要把示例流程编码写入业务代码、测试或文档。

调用前遵循 [临时 API 认证](authentication.md)。没有有效认证时，不请求流程元数据，改为向用户请求当前会话的临时令牌。

响应 `data` 的主要字段：

~~~text
processCode, processName, processVersion
tables[]:
  tableKey, tableName, dataSourceCode, tableDescription, repeatable
  fields[]: fieldName, fieldType, fieldDescription, primaryKey
~~~

`repeatable=true` 表示明细/可重复表。列表和表单必须保留主表与明细表的关系，不应把所有字段平铺为单一主表。

## 固定流程字段

- `itemid`：`long` 类型，必须存在，且每张流程数据表只能以它作为唯一主键。
- `taskid`：`long` 类型，必须存在，由流程引擎写入；它不是主键，业务代码和表单不得主动赋值。

流程绑定表不满足上述约定，或元数据结果与此冲突时，停止实现并请用户先确认表结构或流程配置。后续将绑定流程的新表也必须先定义这两个字段。

## 使用规则

1. 将返回的 `tableName`、字段类型、主键和字段描述作为表单与列表的事实来源。
2. 把表映射为已有表，不使用 AutoTable，不创建或删除列。
3. 如果返回为空、表字段与需求冲突、需要数据库视图，或需要新增/变更字段，向用户逐项确认后再进行下一步；不要猜测。
4. 流程表绑定和默认表单绑定分别使用 Server API；具体写入约束见 [流程绑定配置](process-bindings.md)。
5. 该能力不包含流程图制作、待办/审批页面或流程流转实现。
