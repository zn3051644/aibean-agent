# 流程绑定表元数据

## 查询

当且仅当用户已给出流程编码时，先调用认证后的 Biz API：

```text
GET /bapi/process/meta/bound-fields?processCode=<流程编码>
```

可选参数 `processVersion` 为空时表示读取命中的流程版本。不要把示例流程编码写入业务代码、测试或文档。

响应 `data` 的主要字段：

```text
processCode, processName, processVersion
tables[]:
  tableKey, tableName, dataSourceCode, tableDescription, repeatable
  fields[]: fieldName, fieldType, fieldDescription, primaryKey
```

`repeatable=true` 表示明细/可重复表。列表和表单必须保留主表与明细表的关系，不应把所有字段平铺为单一主表。

## 使用规则

1. 将返回的 `tableName`、字段类型、主键和字段描述作为表单与列表的事实来源。
2. 把表映射为已有表，不使用 AutoTable，不创建或删除列。
3. 如果返回为空、表字段与需求冲突、需要数据库视图，或需要新增/变更字段，向用户逐项确认后再进行下一步；不要猜测。
4. 该能力只解决元数据查询。流程图制作和表单路径绑定仍由用户在流程设计器完成。
