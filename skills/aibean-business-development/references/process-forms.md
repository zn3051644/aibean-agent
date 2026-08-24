# 流程业务表单

## 文件位置和加载方式

本地源码开发只直接读写前端仓库：

```text
public/form/forms/<relative-path>.vue
```

不要调用 `bapi/form/file/save`。`src/views/form/formvue/vform.vue` 在本地模式会按表单相对路径动态加载该文件；交付时报告的路径不带 `.vue` 后缀，例如 `expense/ExpenseApply`。

## 创建方式

1. 从 `public/form/forms/templete.vue` 复制最接近的基础结构，再按流程元数据替换表名、字段、标签和校验。
2. `public/form/formConfig.json` 是已注册 `yd-*` 组件及其文件位置的清单；只使用其中已有组件，或先遵循现有组件扩展方式。
3. 字段组件位于 `public/form/components/field/`，布局组件位于 `public/form/components/layout/`。优先复用它们，而不是以裸 Element Plus 控件绕过字段的可读/可写控制。

## 必须遵守的表单协议

- 具体表单通过 `defineModel('formData')` 接收数据，并接收 `formSchema` 与可选 `formContext`。
- 主表字段使用 `formData.<tableName>.<fieldName>` 和 `formSchema.<tableName>.<fieldName>`；显示条件保留 `readable`，组件会基于 `writeable` 控制禁用状态。
- 明细表使用 `yd-table`、对应数组 `formData.<tableName>` 与 `formCore.Grid`；主键、`RelationRowGuid` 和 `RelationParentRowGuid` 的关系不可删除或自行重命名。
- 校验写在 `el-form-item` 的 `prop` 与规则中，名称必须与实际表/字段完全一致。
- 如果表单实现了额外业务校验，可通过 `defineExpose({ validateForm })` 让运行壳调用。不要重写运行壳的流程提交、审批记录或任务处理能力。
