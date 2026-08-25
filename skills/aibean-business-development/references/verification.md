# 验证清单

按实际改动范围执行，不要为技能文档变更或单一小改动默认跑全套测试。

## 后端

- 至少编译受影响的 Maven 模块；在标准结构中优先使用根目录的 `mvn -pl aibean-biz-web -am compile`。
- 新表使用 AutoTable 时，不要替用户直接访问数据库验证。明确提示用户重启 Biz API，并在启动日志中确认建表/更新结果。
- 对既有或流程表，不应出现 AutoTable DDL 变更。

## 前端

- 至少运行与改动相称的检查；项目提供 `pnpm run vue-tsc`、`pnpm run build` 和 `pnpm run lint:oxlint`。
- 列表页核对：查询、重置、分页、远程排序、空态、字段格式化以及 API 错误处理。
- 表单核对：相对路径可加载、表名/字段名与元数据匹配、只读/可写状态、必填校验和明细行增删。
- 流程表核对：每张表均有 `long` 类型的 `itemid` 与 `taskid`；仅 `itemid` 是主键，业务代码和可编辑表单不写入 `taskid`。

## 在线流程配置

- 流程元数据必须来自 `GET /api/process/meta/bound-fields`，而不是旧的 BAPI 代理。
- 每次 `POST /api/process/initiator-table/operate` 或 `POST /api/process/default-form/update` 前，记录已展示请求体并取得当次用户确认；令牌与其值不得写入任何记录。
- 表绑定写入后重新查询流程元数据，核对表名、主表/明细表关系和字段；默认表单写入后核对接口成功响应与非空相对路径。只有用户明确要求清空时才允许发送空 `defaultForm`。

## 完成交接

报告执行过的命令及结果；未执行的检查必须说明原因。流程表单还需报告其相对路径，以及已执行或待用户确认的默认表单绑定状态。
