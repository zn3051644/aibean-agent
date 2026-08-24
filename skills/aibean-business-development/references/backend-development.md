# Java Biz API 开发

## 模块与包

业务代码统一进入 `aibean-biz-web`。当前仓库没有通用业务 CRUD 样例时，以业务域聚合而非按技术层散落为默认结构，例如：

```text
aibean-biz-web/src/main/java/com/aibean/biz/web/business/<domain>/
├── controller/
├── dto/
├── entity/
├── mapper/
├── service/
└── vo/
```

如果用户工作区已存在某业务域的包结构，先遵循它。Controller 使用 Spring Web、`@Tag` / `@Operation`，请求 DTO 使用 Jakarta Validation。响应使用 `com.aibean.biz.common.result.R`，业务校验失败使用项目现有的 `BizException` / `ResultCode` 约定；不要自造另一套响应格式。

## 数据访问

- 使用 MyBatis-Flex 实体、Mapper（`BaseMapper<T>`）和现有查询方式。主键、时间字段、字段注释与数据库列名要明确映射。
- 用户明确确认的全新业务表：实体可使用 `@AutoTable`、`@AutoColumn`、`@Table`、`@Id` / `@PrimaryKey` 等现有写法，由用户重启 Biz API 执行 AutoTable 建表。
- 已有表或流程绑定表：只使用 `@Table` / `@Column` 等映射，禁止 `@AutoTable`，禁止用 Java 实体影响其结构。
- 当前应用配置可能设置 `auto-table.mode: update` 与 `auto-drop-column: true`。删除或收窄 AutoTable 字段可能损坏数据；除非用户对具体字段和影响明确确认，不得做此类变更。

## 设计准则

将分页、筛选和排序 DTO 设计为能直接服务 `vxe-grid` 的稳定 API。只暴露页面所需字段；不要把实体、SDK 异常或底座内部实现直接泄露给前端。读取组织、流程或其他底座数据时，复用已有 Biz 代理/SDK 能力，而不是绕过认证直连 server API。
