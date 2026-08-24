# 前端列表与业务页面

先查看用户工作区中最相近的页面；没有更贴近的实现时，使用 `src/views/panel/demo/demoApp.vue` 作为列表页基线。

## 列表页面基线

- 页面使用 `page-header`、`query-custom-panel` 与 `vxe-grid`。
- 查询条件保存在 `queryForm`，查询条件定义在 `queryParams`；刷新与重置通过 `queryRef.value.getQueryFilters()` 和 `tableRef.value.commitProxy('reload')`。
- `vxe-grid` 的 `proxyConfig` 开启 `seq` 和远程排序，并将后端分页结果转换为 `{ page: { total }, result: list }`。列名、筛选字段和后端 DTO 字段必须一致；不一致时在页面内维护明确的映射，不能依靠隐式大小写转换。
- 在 `src/api/<业务域>.ts` 使用 `/@/utils/request` 封装每个业务 API。现有调用使用相对地址，如 `bapi/menu/routes`，由 Vite 的 `/bapi` 代理处理；不要硬编码服务主机、端口或令牌。
- 列表应连接真实 API，删除 `demoApp.vue` 中用于演示的 mock 数据和延迟查询模式。

## 页面与流程的边界

待办、审批、提交、驳回和流程图由既有页面处理，业务开发不复制这些页面。

若需求明确需要从业务列表发起已有流程，可复用 `/@/utils/formManager` 的 `openPostWindow(processCode, options)`；它仅作为入口，不包含流程设计或流转实现。

## 动态菜单

新业务页面通常不手工登记前端静态路由。菜单由 Biz API 创建并由 `bapi/menu/routes` 返回动态路由。创建前读取 [菜单约定](menus.md)。
