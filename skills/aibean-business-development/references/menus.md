# 动态菜单

菜单由 Biz API 持久化，前端通过 `bapi/menu/routes` 获取动态路由。

所有菜单 API 都需要当前会话认证。调用前遵循 [临时 API 认证](authentication.md)；认证失败时不重试写入，也不尝试绕过菜单 API。

## 安全写入流程

1. 先调用 `GET /bapi/menu/admin/tree`（需要时带 `groupCode`）读取当前结构。
2. 根据同类菜单确定父级、`code`、`routeName`、`path`、`component`、图标和排序。不要覆盖不相关菜单。
3. 写入前只向用户确认菜单名称。确认后通过前端已有 `SaveMenu` 封装或同等的认证调用写入 `POST /bapi/menu/admin/save`。

Vue 菜单至少需要 `code`、`title`、`pageType: 'VUE'`、`path` 和 `component`；`code` 必须唯一。组件路径应与实际页面一致，例如 `@/views/<domain>/<Page>.vue`。不要额外实现安全组菜单授权，除非用户另行要求。
